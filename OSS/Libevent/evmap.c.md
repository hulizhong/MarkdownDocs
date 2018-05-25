## ReadMe
evmap相关概念；

## evmpa\_io\_active\_
把一组事件激活（等待在fd上的、注册到base上的）
```cpp
void evmap_io_active_(struct event_base *base, evutil_socket_t fd, short events)
{
	struct event_io_map *io = &base->io;
	struct evmap_io *ctx;
	struct event *ev;

#ifndef EVMAP_USE_HT
	if (fd < 0 || fd >= io->nentries)
		return;
#endif
	GET_IO_SLOT(ctx, io, fd, evmap_io);

	if (NULL == ctx)
		return;
	LIST_FOREACH(ev, &ctx->events, ev_io_next) {
		if (ev->ev_events & events)
			event_active_nolock_(ev, ev->ev_events & events, 1);
	}
}
```
