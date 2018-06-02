
## TServer,TServerFramework
记录thrift server的基础类，如下：
[server::TServer](#TServer)
[server::TServerFramework](#TServerFramework)


### TServer
开始封装thrift server需要的组件，如processor, transport..；
不能实例化的一个类；
```cpp
class TServer : public concurrency::Runnable {
public:
  virtual ~TServer() {}

  virtual void serve() = 0; //替代Runnable的run()

  virtual void stop() {}

  // Allows running the server as a Runnable thread
  virtual void run() { serve(); }

  ... get the processor, transport, protocol, eventHandler.

protected:
  ... Constructor ..

   // Get a TProcessor to handle calls on a particular connection.
   //
   // This method should only be called once per connection (never once per
   // call).  This allows the TProcessorFactory to return a different processor
   // for each connection if it desires.
  boost::shared_ptr<TProcessor> getProcessor(boost::shared_ptr<TProtocol> inputProtocol,
                                             boost::shared_ptr<TProtocol> outputProtocol,
                                             boost::shared_ptr<TTransport> transport) {
    TConnectionInfo connInfo;
    connInfo.input = inputProtocol;
    connInfo.output = outputProtocol;
    connInfo.transport = transport;
    return processorFactory_->getProcessor(connInfo);
  }

  // Class variables
  boost::shared_ptr<TProcessorFactory> processorFactory_;
  boost::shared_ptr<TServerTransport> serverTransport_; //相当于server的socket；

  boost::shared_ptr<TTransportFactory> inputTransportFactory_;
  boost::shared_ptr<TTransportFactory> outputTransportFactory_;

  boost::shared_ptr<TProtocolFactory> inputProtocolFactory_;
  boost::shared_ptr<TProtocolFactory> outputProtocolFactory_;

  boost::shared_ptr<TServerEventHandler> eventHandler_; //事件回调；

public:
  ... set transportFactory, protocolFactory, eventHandler.
};
```

### TServerFramework 
重要函数如下：
[TServerFramework::serve](#TServerFramework::serve)
[TServerFramework::newlyConnectedClient](#TServerFramework::newlyConnectedClient)
[TServerFramework::disposeConnectedClient](#TServerFramework::disposeConnectedClient)

TSserverFrameWork为所有的thrift server提供基础的服务；
```cpp
class TServerFramework : public TServer {
public:
  .. construct function. ..

	//从TServerTransport接收client，并处理它们；
  virtual void serve();

	//Interrupt serve() so that it meets post-conditions and returns.
  virtual void stop();

	//返回限制的客户端个数；
  virtual int64_t getConcurrentClientLimit() const;

	//返回当前连接上来的客户端个数；
  virtual int64_t getConcurrentClientCount() const;

  virtual int64_t getConcurrentClientCountHWM() const;

  virtual void setConcurrentClientLimit(int64_t newLimit);

protected:
	//处理连接上来的client
  virtual void onClientConnected(const boost::shared_ptr<TConnectedClient>& pClient) = 0;

	//处理关闭的client
  virtual void onClientDisconnected(TConnectedClient* pClient) = 0;

private:
	//处理刚连接上来的clients，如果连接数达到上限此调用会阻塞；
  void newlyConnectedClient(const boost::shared_ptr<TConnectedClient>& pClient);

	//处理连接关闭的client；调用onClientDisconnected，并删除client；
  void disposeConnectedClient(TConnectedClient* pClient);

	//条件变量，用于控制当前clients数量；
  apache::thrift::concurrency::Monitor mon_;

	//当前clients数量；
  int64_t clients_;

	//并发clents的高水位；hwm_ = std::max(hwm_, clients_);好奇怪？？rwhy
  int64_t hwm_;

	//当前并发client的限制；如果大于此，那么会触发mon_.wait()一个或多个client释放；
  int64_t limit_;
};
```

#### TServerFramework::serve
在一个死循环里面接收请求、并对连接上来的client进行处理；（处理函数未实现需要子类进行实现，可以是异步吗？？？rwhy）
处理完之后，再接收下一个请求；
```cpp
void TServerFramework::serve() {
  shared_ptr<TTransport> client;
  shared_ptr<TTransport> inputTransport;
  shared_ptr<TTransport> outputTransport;
  shared_ptr<TProtocol> inputProtocol;
  shared_ptr<TProtocol> outputProtocol;

  // Start the server listening
  serverTransport_->listen();

  // Run the preServe event to indicate server is now listening
  // and that it is safe to connect.
  if (eventHandler_) {
    eventHandler_->preServe();
  }

  // Fetch client from server
  for (;;) {
    try {
      // Dereference any resources from any previous client creation
      // such that a blocking accept does not hold them indefinitely.
      outputProtocol.reset();
      inputProtocol.reset();
      outputTransport.reset();
      inputTransport.reset();
      client.reset();

      // If we have reached the limit on the number of concurrent
      // clients allowed, wait for one or more clients to drain before
      // accepting another.
      {
        Synchronized sync(mon_);
        while (clients_ >= limit_) {
          mon_.wait();
        }
      }

      client = serverTransport_->accept();

      inputTransport = inputTransportFactory_->getTransport(client);
      outputTransport = outputTransportFactory_->getTransport(client);
      inputProtocol = inputProtocolFactory_->getProtocol(inputTransport);
      outputProtocol = outputProtocolFactory_->getProtocol(outputTransport);

      newlyConnectedClient(shared_ptr<TConnectedClient>(
          new TConnectedClient(getProcessor(inputProtocol, outputProtocol, client),
                               inputProtocol,
                               outputProtocol,
                               eventHandler_,
                               client),
          bind(&TServerFramework::disposeConnectedClient, this, _1))); //自定义shared_ptr的删除器；

    } catch (TTransportException& ttx) {
      releaseOneDescriptor("inputTransport", inputTransport);
      releaseOneDescriptor("outputTransport", outputTransport);
      releaseOneDescriptor("client", client);
      if (ttx.getType() == TTransportException::TIMED_OUT) {
        // Accept timeout - continue processing.
        continue;
      } else if (ttx.getType() == TTransportException::END_OF_FILE
                 || ttx.getType() == TTransportException::INTERRUPTED) {
        // Server was interrupted.  This only happens when stopping.
        break;
      } else {
        // All other transport exceptions are logged.
        // State of connection is unknown.  Done.
        string errStr = string("TServerTransport died: ") + ttx.what();
        GlobalOutput(errStr.c_str());
        break;
      }
    }
  }

  releaseOneDescriptor("serverTransport", serverTransport_);
}
```

#### TServerFramework::newlyConnectedClient
客户端连接上边之后，调用此函数进行处理；
```cpp
void TServerFramework::newlyConnectedClient(const boost::shared_ptr<TConnectedClient>& pClient) {
	//server是否是阻塞服务（一个服务完之后再服务下一个client），就看这个函数是否是阻塞的了；
  onClientConnected(pClient); //这是个接口函数，未实现，继承类需要实现它；

  // Count a concurrent client added.
  Synchronized sync(mon_);
  ++clients_;
  hwm_ = std::max(hwm_, clients_);
}
```

#### TServerFramework::disposeConnectedClient
```cpp
void TServerFramework::disposeConnectedClient(TConnectedClient* pClient) {
  {
    // Count a concurrent client removed.
    Synchronized sync(mon_);
    if (limit_ - --clients_ > 0) {
      mon_.notify();
    }
  }
  onClientDisconnected(pClient);
  delete pClient;
}
```

## SimpleServer  
单线程，每个线程阻塞
```cpp
class TSimpleServer : public TServerFramework {
public:
  .. construct function ..

protected:
  virtual void onClientConnected(const boost::shared_ptr<TConnectedClient>& pClient) /* override */;
  virtual void onClientDisconnected(TConnectedClient* pClient) /* override */;

private:
  void setConcurrentClientLimit(int64_t newLimit); // hide
};
```
重要成员函数如下：
[TSimpleServer::onClientConnected](#TSimpleServer::onClientConnected)
[TSimpleServer::onClientDisconnected](#TSimpleServer::onClientDisconnected)

### TSimpleServer::onClientConnected
直接调用client对象的run函数
```cpp
void TSimpleServer::onClientConnected(const shared_ptr<TConnectedClient>& pClient) {
  pClient->run();
}
```

### TSimpleServer::onClientDisconnected
空实现
```cpp
void TSimpleServer::onClientDisconnected(TConnectedClient*) {
}
```


## ThreadServer  
多线程，来一个连接开一个线程，线程阻塞。
在TSimpleServer基础上多了个ThreadFactory组件；
```cpp
class TThreadedServer : public TServerFramework {
public:
  .. constructor ..

  virtual ~TThreadedServer();

  /**
   * Post-conditions (return guarantees):
   *   There will be no clients connected.
   */
  virtual void serve();

protected:
  virtual void onClientConnected(const boost::shared_ptr<TConnectedClient>& pClient) /* override */;
  virtual void onClientDisconnected(TConnectedClient* pClient) /* override */;

	//多出来的，线程工厂类；
  boost::shared_ptr<apache::thrift::concurrency::ThreadFactory> threadFactory_;
  apache::thrift::concurrency::Monitor clientsMonitor_;
};
```

重要的函数如下：
[TThreadedServer::serve](#TThreadedServer::serve)
[TThreadedServer::onClientConnected](#TThreadedServer::onClientConnected)
[TThreadedServer::onClientDisconnected](#TThreadedServer::onClientDisconnected)

### TThreadedServer::serve
处理流程还是沿用[TServerFramework::server](#TServerFramework::serve)
只是serve退出的时候，要判断所有的client都处理完了，方可退出； --rwhy
```cpp
void TThreadedServer::serve() {
  TServerFramework::serve(); //调用基类的serve服务；

  // Drain all clients - no more will arrive
  try {
    Synchronized s(clientsMonitor_);
    while (getConcurrentClientCount() > 0) {
      clientsMonitor_.wait();
    }
  } catch (TException& tx) {
    string errStr = string("TThreadedServer: Exception joining workers: ") + tx.what();
    GlobalOutput(errStr.c_str());
  }
}
```

### TThreadedServer::onClientConnected
```cpp
void TThreadedServer::onClientConnected(const shared_ptr<TConnectedClient>& pClient) {
	//创建一个线程来运行pClient这个runnable.
	//不会阻塞，只是创建线程，而在线程函数里才会运行cClient->run()
  threadFactory_->newThread(pClient)->start();
}
```

### TThreadedServer::onClientDisconnected
```cpp
void TThreadedServer::onClientDisconnected(TConnectedClient* pClient) {
  THRIFT_UNUSED_VARIABLE(pClient);
  Synchronized s(clientsMonitor_);
  if (getConcurrentClientCount() == 0) {
    clientsMonitor_.notify();
  }
}
```



## ThreadPoolServer  
线程池，每个线程阻塞



## NonBlockingServer  
线程池，非阻塞。用epoll模型实现。用起来很方便。


NoStarveReadWriteMutex for what ??  


## TConnectedClient 
代表一个已连接上服务器的客户端
```cpp
class TConnectedClient : public apache::thrift::concurrency::Runnable {
public:
   //Constructor.
   //Destructor.

   //Drive the client until it is done.
   //The client processing loop is:
   //
   //[optional] call eventHandler->createContext once
   //[optional] call eventHandler->processContext per request
   //           call processor->process per request
   //             handle expected transport exceptions:
   //               END_OF_FILE means the client is gone
   //               INTERRUPTED means the client was interrupted
   //                           by TServerTransport::interruptChildren()
   //             handle unexpected transport exceptions by logging
   //             handle standard exceptions by logging
   //             handle unexpected exceptions by logging
   //           cleanup()
  virtual void run() /* override */;

protected:
   //Cleanup after a client.  This happens if the client disconnects,
   //or if the server is stopped, or if an exception occurs.
   //
   //The cleanup processing is:
   //[optional] call eventHandler->deleteContext once
   //           close the inputProtocol's TTransport
   //           close the outputProtocol's TTransport
   //           close the client
  virtual void cleanup();

private:
  boost::shared_ptr<apache::thrift::TProcessor> processor_;
  boost::shared_ptr<apache::thrift::protocol::TProtocol> inputProtocol_;
  boost::shared_ptr<apache::thrift::protocol::TProtocol> outputProtocol_;
  boost::shared_ptr<apache::thrift::server::TServerEventHandler> eventHandler_;
  boost::shared_ptr<apache::thrift::transport::TTransport> client_; //代表client本身；

   //Context acquired from the eventHandler_ if one exists.
  void* opaqueContext_;
};
```
重要的函数如下：
[TConnectedClient::run](#TConnectedClient::run)
[TConnectedClient::cleanup](#TConnectedClient::cleanup)

### TConnectedClient::run
调用eventHandler创建会话上下文、处理上下文、处理processor、释放资源；
```cpp
void TConnectedClient::run() {
  if (eventHandler_) {
    opaqueContext_ = eventHandler_->createContext(inputProtocol_, outputProtocol_);
  }

  for (bool done = false; !done;) {
    if (eventHandler_) {
      eventHandler_->processContext(opaqueContext_, client_);
    }

    try {
      if (!processor_->process(inputProtocol_, outputProtocol_, opaqueContext_)) {
        break;
      }
    } catch (const TTransportException& ttx) {
      switch (ttx.getType()) {
      case TTransportException::TIMED_OUT:
        // Receive timeout - continue processing.
        continue;

      case TTransportException::END_OF_FILE:
      case TTransportException::INTERRUPTED:
        // Client disconnected or was interrupted.  No logging needed.  Done.
        done = true;
        break;

      default: {
        // All other transport exceptions are logged.
        // State of connection is unknown.  Done.
        string errStr = string("TConnectedClient died: ") + ttx.what();
        GlobalOutput(errStr.c_str());
        done = true;
        break;
      }
      }
    } catch (const TException& tex) {
      string errStr = string("TConnectedClient processing exception: ") + tex.what();
      GlobalOutput(errStr.c_str());
      // Continue processing
    }
  }

  cleanup();
}
```

### TConnectedClient::cleanup
释放会话上下文、关闭连接；
```cpp
void TConnectedClient::cleanup() {
  if (eventHandler_) {
    eventHandler_->deleteContext(opaqueContext_, inputProtocol_, outputProtocol_);
  }

  try {
    inputProtocol_->getTransport()->close();
  } catch (const TTransportException& ttx) {
    string errStr = string("TConnectedClient input close failed: ") + ttx.what();
    GlobalOutput(errStr.c_str());
  }

  try {
    outputProtocol_->getTransport()->close();
  } catch (const TTransportException& ttx) {
    string errStr = string("TConnectedClient output close failed: ") + ttx.what();
    GlobalOutput(errStr.c_str());
  }

  try {
    client_->close();
  } catch (const TTransportException& ttx) {
    string errStr = string("TConnectedClient client close failed: ") + ttx.what();
    GlobalOutput(errStr.c_str());
  }
}
```
