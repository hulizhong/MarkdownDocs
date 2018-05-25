[toc]

## ReadMe

## signal的eventop
evsigops 
```cpp
static const struct eventop evsigops = {
	"signal",
	NULL,
	evsig_add,
	evsig_del,
	NULL,
	NULL,
	0, 0, 0
};
```

## evsig\_add
```cpp
static int
evsig_add(struct event_base *base, evutil_socket_t evsignal, short old, short events, void *p)
{
	struct evsig_info *sig = &base->sig;
	(void)p;

	EVUTIL_ASSERT(evsignal >= 0 && evsignal < NSIG);

	/* catch signals if they happen quickly */
	EVSIGBASE_LOCK();
	if (evsig_base != base && evsig_base_n_signals_added) {
		event_warnx("Added a signal to event base %p with signals "
		    "already added to event_base %p.  Only one can have "
		    "signals at a time with the %s backend.  The base with "
		    "the most recently added signal or the most recent "
		    "event_base_loop() call gets preference; do "
		    "not rely on this behavior in future Libevent versions.",
		    base, evsig_base, base->evsel->name);
	}
	evsig_base = base;
	evsig_base_n_signals_added = ++sig->ev_n_signals_added;
	evsig_base_fd = base->sig.ev_signal_pair[1];
	EVSIGBASE_UNLOCK();

	event_debug(("%s: %d: changing signal handler", __func__, (int)evsignal));
	if (evsig_set_handler_(base, (int)evsignal, evsig_handler) == -1) {
		goto err;
	}


	if (!sig->ev_signal_added) {
		if (event_add_nolock_(&sig->ev_signal, NULL, 0))
			goto err;
		sig->ev_signal_added = 1;
	}

	return (0);

err:
	EVSIGBASE_LOCK();
	--evsig_base_n_signals_added;
	--sig->ev_n_signals_added;
	EVSIGBASE_UNLOCK();
	return (-1);
}
```

## evsig\_del
```cpp
static int
evsig_del(struct event_base *base, evutil_socket_t evsignal, short old, short events, void *p)
{
	EVUTIL_ASSERT(evsignal >= 0 && evsignal < NSIG);

	event_debug(("%s: "EV_SOCK_FMT": restoring signal handler",
		__func__, EV_SOCK_ARG(evsignal)));

	EVSIGBASE_LOCK();
	--evsig_base_n_signals_added;
	--base->sig.ev_n_signals_added;
	EVSIGBASE_UNLOCK();

	return (evsig_restore_handler_(base, (int)evsignal));
}
```

