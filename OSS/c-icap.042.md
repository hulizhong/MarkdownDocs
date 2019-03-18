

[TOC]



## ReadMe

c-icap 0.4.2 概要源代码分析。



## 概要

### 并发模型

多进程模型：类似apache有个父进程对各子进程进行健康检查、子进程的扩展、收缩；各子进程内部创建指定工作线程进行对外服务。
单进程模型：只有一个进程，在这个进程中创建指定数工作线程，直接对外服务。---没有健康检查这一块了？



### 连接建立、处理

每个进程有一个接收请求的线程，接收完了之后放入连接队列（队列的长度等于进程内工作线程数）中，完了进程内部的工作线程从队列中取连接进行服务。一个生产者、多消费者模型。



## aserver.c 主函数

main

```cpp
int main(int argc, char **argv)
{
#if ! defined(_WIN32)
     __log_error = (void (*)(void *, const char *,...)) log_server;     /*set c-icap library log  function */
#else
     __vlog_error = vlog_server;        /*set c-icap library  log function */
#endif

     mem_init();
     init_internal_lookup_tables();
     ci_acl_init();
     init_http_auth();
     if (init_body_system() != CI_OK) {
         ci_debug_printf(1, "Can not initialize body system\n");
         exit(-1);
     }
     ci_txt_template_init();
     ci_txt_template_set_dir(DATADIR"templates");
     commands_init();

     if (!(CI_CONF.MAGIC_DB = ci_magic_db_load(CI_CONF.magics_file))) {
          ci_debug_printf(1, "Can not load magic file %s!!!\n",
                          CI_CONF.magics_file);
     }
     init_conf_tables();
     request_stats_init();
     init_modules();
     init_services();
     config(argc, argv);
     compute_my_hostname();
     ci_debug_printf(2, "My hostname is:%s\n", MY_HOSTNAME);

     if (!log_open()) {
          ci_debug_printf(1, "Can not init loggers. Exiting.....\n");
          exit(-1);
     }

#if ! defined(_WIN32)
     if (is_icap_running(CI_CONF.PIDFILE)) {
          ci_debug_printf(1, "c-icap server already running!\n");
          exit(-1);
     }
     if (DAEMON_MODE)
          run_as_daemon();
     if (!set_running_permissions(CI_CONF.RUN_USER, CI_CONF.RUN_GROUP))
          exit(-1);
     store_pid(CI_CONF.PIDFILE);
#endif

     if (!init_server(CI_CONF.ADDRESS, CI_CONF.PORT, &(CI_CONF.PROTOCOL_FAMILY)))
          return -1;
     post_init_modules();
     post_init_services();
     start_server();
     clear_pid(CI_CONF.PIDFILE);
     return 0;
}

```





## Main Process. 主进程



### start_server

启动主进程

```cpp
int start_server()
{
     int child_indx, pid, i, ctl_socket;
     int childs, freeservers, used, maxrequests, ret;
     char command_buffer[COMMANDS_BUFFER_SIZE];
     int user_informed = 0;

     ctl_socket = ci_named_pipe_create(CI_CONF.COMMANDS_SOCKET);
     if (ctl_socket < 0) {
          ci_debug_printf(1,
                          "Error opening control socket %s: %s. Fatal error, exiting!\n",
                          strerror(errno),
                          CI_CONF.COMMANDS_SOCKET);
          exit(0);
     }

     if (!ci_proc_mutex_init(&accept_mutex, "accept")) {
          ci_debug_printf(1,
                          "Can't init mutex for accepting conenctions. Fatal error, exiting!\n");
          exit(0);
     }
     childs_queue = malloc(sizeof(struct childs_queue));
     if (!create_childs_queue(childs_queue, 2 * CI_CONF.MAX_SERVERS)) {
          ci_proc_mutex_destroy(&accept_mutex);
          ci_debug_printf(1,
                          "Can't init shared memory. Fatal error, exiting!\n");
          exit(0);
     }

     init_commands();
     pid = 1;
#ifdef MULTICHILD  //多进程模式，走这个分支。
     if (CI_CONF.START_SERVERS > CI_CONF.MAX_SERVERS)
          CI_CONF.START_SERVERS = CI_CONF.MAX_SERVERS;

     for (i = 0; i < CI_CONF.START_SERVERS; i++) {
          if (pid)
               pid = start_child(LISTEN_SOCKET);  //创建一堆的子进程。
     }
     if (pid != 0) {  //最后一个子进程创建成功。----意味着所有子进程都创建好了。
          main_signals();

          while (1) {
               if ((ret = wait_for_commands(ctl_socket, command_buffer, 1)) > 0) {
                    ci_debug_printf(5, "I received the command: %s\n",
                                    command_buffer);
                    handle_monitor_process_commands(command_buffer);
               }
               if (ret < 0) {  /*Eof received on pipe. Going to reopen ... */
                    ci_named_pipe_close(ctl_socket);
                    ctl_socket = ci_named_pipe_open(CI_CONF.COMMANDS_SOCKET);
                    if (ctl_socket < 0) {
                         ci_debug_printf(1,
                                         "Error opening control socket. We are unstable and going down!");
                         c_icap_going_to_term = 1;
                    }
               }

               if (c_icap_going_to_term)
                    break;
               childs_queue_stats(childs_queue, &childs, &freeservers, &used,
                                  &maxrequests);
               ci_debug_printf(10,
                               "Server stats: \n\t Children: %d\n\t Free servers: %d\n"
                               "\tUsed servers:%d\n\tRequests served:%d\n",
                               childs, freeservers, used, maxrequests);
               if (MAX_REQUESTS_PER_CHILD > 0 && (child_indx =
                                                  find_a_child_nrequests
                                                  (childs_queue,
                                                   MAX_REQUESTS_PER_CHILD)) >=
                   0) {
                    ci_debug_printf(8,
                                    "Max requests reached for child :%d of pid :%d\n",
                                    child_indx,
                                    childs_queue->childs[child_indx].pid);
                    pid = start_child(LISTEN_SOCKET);
                    //         usleep(500);
                    childs_queue->childs[child_indx].father_said = GRACEFULLY;
                    /*kill a server ... */
                    kill(childs_queue->childs[child_indx].pid, SIGTERM);

               }
               else if ((freeservers <= CI_CONF.MIN_SPARE_THREADS && childs < CI_CONF.MAX_SERVERS)
                        || childs < CI_CONF.START_SERVERS) {
                    ci_debug_printf(8,
                                    "Free Servers: %d, children: %d. Going to start a child .....\n",
                                    freeservers, childs);
                    pid = start_child(LISTEN_SOCKET);
               }
               else if (freeservers >= CI_CONF.MAX_SPARE_THREADS &&
                        childs > CI_CONF.START_SERVERS &&
                        (freeservers - CI_CONF.THREADS_PER_CHILD) > CI_CONF.MIN_SPARE_THREADS) {

                    if ((child_indx = find_an_idle_child(childs_queue)) >= 0) {
                         childs_queue->childs[child_indx].father_said =
                             GRACEFULLY;
                         ci_debug_printf(8,
                                         "Free Servers: %d, children: %d. Going to stop child %d(index: %d)\n",
                                         freeservers, childs,
                                         childs_queue->childs[child_indx].pid,
                                         child_indx);
                         /*kill a server ... */
                         kill(childs_queue->childs[child_indx].pid, SIGTERM);
			 user_informed = 0;
                    }
               }
               else if (childs == CI_CONF.MAX_SERVERS && freeservers < CI_CONF.MIN_SPARE_THREADS) {
		 if(! user_informed) {
		         ci_debug_printf(1,
					 "ATTENTION!!!! Not enough available servers (children %d, free servers %d, used servers %d)!!!!! "
					 "Maybe you should increase the MaxServers and the "
					 "ThreadsPerChild values in c-icap.conf file!!!!!!!!!",childs , freeservers, used);
			 user_informed = 1;
		 }
               }
               if (c_icap_going_to_term)
                    break;
               check_for_exited_childs();
               if (c_icap_reconfigure) {
                    c_icap_reconfigure = 0;
                    if (!server_reconfigure()) {
			ci_debug_printf(1, "Error while reconfiguring, exiting!\n");
			 break;
		    }
               }
          }
          /*Main process exit point */
          ci_debug_printf(1,
                          "Possibly a term signal received. Monitor process going to term all children\n"); //rabin. step2.
          kill_all_childs();
	  system_shutdown();
	  ci_debug_printf(1, "Exiting....\n");
          return 1;
     }
#else
     child_data = (child_shared_data_t *) malloc(sizeof(child_shared_data_t));
     child_data->pid = 0;
     child_data->freeservers = CI_CONF.THREADS_PER_CHILD;
     child_data->usedservers = 0;
     child_data->requests = 0;
     child_data->connections = 0;
     child_data->to_be_killed = 0;
     child_data->father_said = 0;
     child_data->idle = 1;
     child_data->stats_size = ci_stat_memblock_size();
     child_data->stats = malloc(child_data->stats_size);
     child_data->stats->sig = MEMBLOCK_SIG;
     ci_stat_attach_mem(child_data->stats, child_data->stats_size, NULL);
     child_main(LISTEN_SOCKET, 0);
     ci_proc_mutex_destroy(&accept_mutex);
     destroy_childs_queue(childs_queue);
#endif
     return 1;
}

```





### start_child

创建子进程。

```cpp
int start_child(int fd)
{
     int pid;
     int pfd[2];
     int children_num, free_servers, used_servers, max_requests;

     if (pipe(pfd) < 0) {
          ci_debug_printf(1,
                          "Error creating pipe for communication with child\n");
          return -1;
     }
     if (fcntl(pfd[0], F_SETFL, O_NONBLOCK) < 0
         || fcntl(pfd[1], F_SETFL, O_NONBLOCK) < 0) {
          ci_debug_printf(1, "Error making the child pipe non-blocking\n");
          close(pfd[0]);
          close(pfd[1]);
     }
     if ((pid = fork()) == 0) { //A Child .......
          MY_PROC_PID = getpid();
          if (!attach_childs_queue(childs_queue)) {
              ci_debug_printf(1, "Can not access shared memory for %d child\n", (int)MY_PROC_PID);
              exit(-2);
          }
          child_data =
              register_child(childs_queue, getpid(), CI_CONF.THREADS_PER_CHILD, pfd[1]);
          if (!child_data) {
              childs_queue_stats(childs_queue, &children_num, &free_servers,
                                 &used_servers, &max_requests);
              ci_debug_printf(1, "Not available slot in shared memory for %d child (Number of slots: %d, Running children: %d, Free servers: %d, Used servers: %d)!\n", (int)MY_PROC_PID,
                              childs_queue->size,
                              children_num,
                              free_servers,
                              used_servers
                  );
              exit(-3);
          }
          close(pfd[1]);
          child_main(fd, pfd[0]);
          exit(0);
     }
     else {
          close(pfd[0]);
          announce_child(childs_queue, pid);
          return pid;
     }
}
```







### c_icap_going_to_term = 1.

```cpp
static void sigint_handler_main(int sig)
{
     if (sig == SIGTERM) {
     }
     else if (sig == SIGINT) {
     }
     else {
     }

     c_icap_going_to_term = 1;
}
//SIGINT, SIGTERM 的回调函数。

void stop_command(const char *name, int type, const char **argv)
{
     c_icap_going_to_term = 1;
}
```







## Child Process. 子进程

### child_main

子进程主函数

```cpp
void child_main(int sockfd, int pipefd)
{
     ci_thread_t thread;
     int i, ret;

     signal(SIGTERM, SIG_IGN);  /*Ignore parent requests to kill us untill we are up and running */
     ci_thread_mutex_init(&threads_list_mtx);
     ci_thread_mutex_init(&counters_mtx);
     ci_thread_cond_init(&free_server_cond);
     
     ci_stat_attach_mem(child_data->stats, child_data->stats_size, NULL);

     threads_list =
         (server_decl_t **) malloc((CI_CONF.THREADS_PER_CHILD + 1) *
                                   sizeof(server_decl_t *));
     con_queue = init_queue(CI_CONF.THREADS_PER_CHILD);

     for (i = 0; i < CI_CONF.THREADS_PER_CHILD; i++) { //创建子线程。
          if ((threads_list[i] = newthread(con_queue)) == NULL) {
               exit(-1);        // FATAL error.....
          }
          ret =
              ci_thread_create(&thread, (void *(*)(void *)) thread_main,
                               (void *) threads_list[i]);
          threads_list[i]->srv_pthread = thread;
     }
     threads_list[CI_CONF.THREADS_PER_CHILD] = NULL;
     /*Now start the listener thread.... */
     ret = ci_thread_create(&thread, (void *(*)(void *)) listener_thread,
                            (void *) &sockfd);
     listener_thread_id = thread;
     
     /*set srand for child......*/
     srand(((unsigned int)time(NULL)) + (unsigned int)getpid());
     /*I suppose that all my threads are up now. We can setup our signal handlers */
     child_signals();

     /* A signal from parent may comes while we are starting.
        Listener will not accept any request in this case, (it checks on 
        the beggining of the accept loop for parent commands) so we can 
        shutdown imediatelly even if the parent said gracefuly.*/
     if (child_data->father_said)
         child_data->to_be_killed = IMMEDIATELY;

     /*start child commands may have non thread safe code but the worker threads
       does not serving requests yet.*/
     commands_execute_start_child();

     /*Signal listener to start accepting requests.*/
     int doStart = 0;
     do {
         ci_thread_mutex_lock(&counters_mtx);
         doStart = listener_running;
         ci_thread_mutex_unlock(&counters_mtx);
         if (!doStart)
             ci_usleep(5);
     } while(!doStart);
     ci_thread_cond_signal(&free_server_cond);

     while (!child_data->to_be_killed) { //从父进程处获取指令执行。
          char buf[512];
          int bytes;
          if ((ret = ci_wait_for_data(pipefd, 1, wait_for_read)) > 0) { /*data input */
               bytes = ci_read_nonblock(pipefd, buf, 511);
               if (bytes == 0) {
                    ci_debug_printf(1,
                                    "Parent closed the pipe connection! Going to term immediately!\n");
                    child_data->to_be_killed = IMMEDIATELY;
               } else {
                    buf[bytes] = '\0';
                    handle_child_process_commands(buf);
               }
          }
          else if (ret < 0) {
               ci_debug_printf(1,
                               "An error occured while waiting for commands from parent. Terminating!\n");
               child_data->to_be_killed = IMMEDIATELY;
          }
          if (!listener_running && !child_data->to_be_killed) {
               ci_debug_printf(1,
                               "Ohh!! something happened to listener thread! Terminating\n");
               child_data->to_be_killed = GRACEFULLY;
          }
          commands_exec_scheduled();
     }

     ci_debug_printf(5, "Child :%d going down :%s\n", getpid(),
                     child_data->to_be_killed == IMMEDIATELY? 
                     "IMMEDIATELY" : "GRACEFULLY"); //rabin. step1.
     
     cancel_all_threads();
     commands_execute_stop_child();
     exit_normaly();
}
```





## Thread. 子进程中的线程

### thread_main. 

```cpp
int thread_main(server_decl_t * srv)
{
     ci_connection_t con;
     char clientname[CI_MAXHOSTNAMELEN + 1];
     int ret, request_status = CI_NO_STATUS;
     int keepalive_reqs;
//***********************
     thread_signals(0);
//*************************
     srv->srv_id = getpid();    //Setting my pid ...

     for (;;) {
          /*
             If we must shutdown IMEDIATELLY it is time to leave the server
             else if we are going to shutdown GRACEFULLY we are going to die 
             only if there are not any accepted connections
           */
          if (child_data->to_be_killed == IMMEDIATELY) {
               srv->running = 0;
               return 1;
          }

          if ((ret = get_from_queue(con_queue, &con)) == 0) {
               if (child_data->to_be_killed) {
                    srv->running = 0;
                    return 1;
               }
               ret = wait_for_queue(con_queue);
               continue;
          }

          if (ret < 0) {        //An error has occured
               ci_debug_printf(1,
                               "Fatal Error!!! Error getting a connection from connections queue!!!\n");
               break;
          }

          ci_thread_mutex_lock(&counters_mtx);  /*Update counters as soon as possible */
          (child_data->freeservers)--;
          (child_data->usedservers)++;
          ci_thread_mutex_unlock(&counters_mtx);

          ci_netio_init(con.fd);
          ret = 1;
          if (srv->current_req == NULL)
               srv->current_req = newrequest(&con);
          else
               ret = recycle_request(srv->current_req, &con);

          if (srv->current_req == NULL || ret == 0) {
               ci_sockaddr_t_to_host(&(con.claddr), clientname,
                                     CI_MAXHOSTNAMELEN);
               ci_debug_printf(1, "Request from %s denied...\n", clientname);
               hard_close_connection((&con));
               goto end_of_main_loop_thread;    /*The request rejected. Log an error and continue ... */
          }

          keepalive_reqs = 0;
          do {
               if (MAX_KEEPALIVE_REQUESTS > 0
                   && keepalive_reqs >= MAX_KEEPALIVE_REQUESTS)
                    srv->current_req->keepalive = 0;    /*do not keep alive connection */
               if (child_data->to_be_killed)    /*We are going to die do not keep-alive */
                    srv->current_req->keepalive = 0;

               if ((request_status = process_request(srv->current_req)) == CI_NO_STATUS) {
                    ci_debug_printf(5,
                                    "Process request timeout or interrupted....\n");
                    ci_request_reset(srv->current_req);
                    break;
               }
               srv->served_requests++;
               srv->served_requests_no_reallocation++;
               keepalive_reqs++;

               /*Increase served requests. I dont like this. The delay is small but I don't like... */
               ci_thread_mutex_lock(&counters_mtx);
               (child_data->requests)++;
               ci_thread_mutex_unlock(&counters_mtx);

               log_access(srv->current_req, request_status);
//             break; //No keep-alive ......

               if (child_data->to_be_killed  == IMMEDIATELY)
                    break;      //Just exiting the keep-alive loop

               /*if we are going to term gracefully we will try to keep our promice for
                 keepalived request....
                */
               if (child_data->to_be_killed  == GRACEFULLY && 
                   srv->current_req->keepalive == 0)
                    break;

               ci_debug_printf(8, "Keep-alive:%d\n",
                               srv->current_req->keepalive);
               if (srv->current_req->keepalive && keepalive_request(srv->current_req)) {
                   ci_debug_printf(8,
                                   "Server %d going to serve new request from client (keep-alive) \n",
                                   srv->srv_id);
               }
               else
                    break;
          } while (1);

          if (srv->current_req) {
               if (request_status != CI_OK || child_data->to_be_killed) {
                    hard_close_connection(srv->current_req->connection);
               }
               else {
                    close_connection(srv->current_req->connection);
               }
          }
          if (srv->served_requests_no_reallocation >
              MAX_REQUESTS_BEFORE_REALLOCATE_MEM) {
               ci_debug_printf(5,
                               "Max requests reached, reallocate memory and buffers .....\n");
               ci_request_destroy(srv->current_req);
               srv->current_req = NULL;
               srv->served_requests_no_reallocation = 0;
          }


        end_of_main_loop_thread:
          ci_thread_mutex_lock(&counters_mtx);
          (child_data->freeservers)++;
          (child_data->usedservers)--;
          ci_thread_mutex_unlock(&counters_mtx);
          ci_thread_cond_signal(&free_server_cond);

     }
     srv->running = 0;
     return 0;
}
```

