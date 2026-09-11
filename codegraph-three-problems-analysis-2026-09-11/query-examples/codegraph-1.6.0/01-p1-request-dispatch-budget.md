# 请求分发链：关键证据与输出预算

- 用例 ID：`p1-request-dispatch-budget`
- 对应问题：P1
- CodeGraph：1.6.0
- Django 项目：`D:\工作空间\harbor\exp\django`
- 记录时间：2026-09-10T12:25:23.857Z
- 用例配置哈希：`早期记录未提供`

## 测试目的

The answer spans the handler and URL resolver, but should not require unrelated Django internals.

## 给 Agent 的任务

```text
Explain the exact runtime flow from BaseHandler.get_response through URL resolution to invocation of the matched view callback. Name the important symbols and files, and distinguish the middleware step from resolver dispatch.
```

## 直接 MCP 查询

### 第 1 次调用

```text
BaseHandler.get_response _get_response resolve_request URLResolver.resolve ResolverMatch callback
```

复现命令：

```powershell
codegraph explore "BaseHandler.get_response _get_response resolve_request URLResolver.resolve ResolverMatch callback" --path "D:\工作空间\harbor\exp\django"
```

| 指标 | 结果 |
|---|---:|
| 响应字符数 | 24944 |
| 估算 token | 8315 |
| 查询耗时 | 717.6 ms |
| 事实覆盖率 | 1 |
| 概念命中 | 6/6 |
| 文件命中 | 2/2 |
| MCP 错误 | 否 |

#### 完整原始返回

````text
**Flow (call path among the symbols you queried)**

1. _get_response (django/core/handlers/base.py:176)
   ↓ calls
2. resolve_request (django/core/handlers/base.py:302)
   ↓ calls
3. resolve (django/urls/resolvers.py:672)

> Full source for these symbols is below — the call flow among them, followed by their bodies.
**Exploration: BaseHandler.get_response _get_response resolve_request URLResolver.resolve ResolverMatch callback**

Found 45 symbols across 3 files.

**Blast radius — what depends on these (update/verify before editing)**

- `URLResolver` (django/urls/resolvers.py:505) — 14 callers in `django/conf/urls/i18n.py`, `django/urls/conf.py`, `django/urls/__init__.py`, `django/views/debug.py` +1 more; tests: `tests/urlpatterns_reverse/tests.py`
- `ResolverMatch` (django/urls/resolvers.py:35) — 5 callers in `django/urls/__init__.py`, `django/urls/resolvers.py`; tests: `tests/urlpatterns_reverse/tests.py`
- `Resolver404` (django/urls/exceptions.py:4) — 14 callers in `django/contrib/admin/sites.py`, `django/contrib/admin/templatetags/admin_urls.py`, `django/urls/base.py`, `django/urls/resolvers.py` +1 more; tests: `tests/urlpatterns/tests.py`, `tests/urlpatterns_reverse/tests.py`
- `resolve_request` (django/core/handlers/base.py:302) — 2 callers in `django/core/handlers/base.py`; no tests found within 3 caller hops
- `BaseHandler` (django/core/handlers/base.py:21) — 1 caller; tests: `django/test/client.py`

**Relationships**

**extends:**
- Resolver404 → Http404
- ASGIHandler → BaseHandler
- WSGIHandler → BaseHandler
- ClientHandler → BaseHandler
- AsyncClientHandler → BaseHandler

**instantiates:**
- i18n_patterns → URLResolver
- _path → URLResolver
- _get_cached_resolver → URLResolver
- get_ns_resolver → URLResolver
- test_no_urls_exception → URLResolver
- test_populate_concurrency → URLResolver
- setUp → URLResolver
- resolve → ResolverMatch
- resolve → ResolverMatch
- resolve → Resolver404
- ... and 17 more

**references:**
- technical_404_response → URLResolver
- test_urlpattern_resolve → ResolverMatch
- resolve → URLPattern
- test_urlpattern_resolve → resolve_test_data
- test_path_trailing_newlines → Resolver404
- test_nonmatching_urls → Resolver404
- test_resolve_value_error_means_no_match → Resolver404
- test_non_regex → Resolver404
- test_404_tried_urls_have_names → Resolver404
- test_invalid_resolve → Resolver404
- ... and 7 more

**calls:**
- resolve → match
- resolve → resolve
- resolve → match
- resolve → _extend_tried
- resolve → get
- resolve → _join_route
- resolve_request → resolve
- test_lazy_route_resolves → resolve
- test_reverse_lazy_object_coercion_by_resolve → resolve
- raises404 → resolve
- ... and 63 more

**Source Code**

> The code below is the **verbatim, current on-disk source** of these files — re-read from disk on this call and line-numbered, byte-for-byte identical to what the Read tool returns. It is NOT a summary, outline, or stale cache. Treat each block as a Read you have already performed: do not Read a file shown here.

**`django/core/handlers/base.py`** — adapt_method_mode(calls), append(calls), set_urlconf(calls), calls(calls), convert_exception_to_response(calls), get_resolver(calls), import_string(calls), adapt_method_mode(method), middleware(calls), ImproperlyConfigured(instantiates), +18 more

```python
27	    def load_middleware(self, is_async=False):
28	        """
29	        Populate middleware lists from settings.MIDDLEWARE.
30	
31	        Must be called after the environment is fixed (see __call__ in

... (gap: adapt_method_mode (django/core/handlers/base.py:106), get_response (django/core/handlers/base.py:138), get_response_async (django/core/handlers/base.py:154)) ...

155	        """
156	        Asynchronous version of get_response.
157	
158	        Funneling everything, including WSGI, into a single async
159	        get_response() is too slow. Avoid the context switch by using
160	        a separate async response path.
161	        """
162	        # Setup default url resolver for this thread.
163	        set_urlconf(settings.ROOT_URLCONF)
164	        response = await self._middleware_chain(request)
165	        response._resource_closers.append(request.close)
166	        if response.status_code >= 400:
167	            await sync_to_async(log_response, thread_sensitive=False)(
168	                "%s: %s",
169	                response.reason_phrase,
170	                request.path,
171	                response=response,
172	                request=request,
173	            )
174	        return response
175	
176	    def _get_response(self, request):
177	        """
178	        Resolve and call the view, then apply view, exception, and
179	        template_response middleware. This method is everything that happens
180	        inside the request/response middleware.
181	        """
182	        response = None
183	        callback, callback_args, callback_kwargs = self.resolve_request(request)
184	
185	        # Apply view middleware
186	        for middleware_method in self._view_middleware:
187	            response = middleware_method(
188	                request, callback, callback_args, callback_kwargs
189	            )
190	            if response:
191	                break
192	
193	        if response is None:
194	            wrapped_callback = self.make_view_atomic(callback)
195	            # If it is an asynchronous view, run it in a subthread.
196	            if iscoroutinefunction(wrapped_callback):
197	                wrapped_callback = async_to_sync(wrapped_callback)
198	            try:
199	                response = wrapped_callback(request, *callback_args, **callback_kwargs)
200	            except Exception as e:
201	                response = self.process_exception_by_middleware(e, request)
202	                if response is None:
203	                    raise
204	
205	        # Complain if the view returned None (a common error).
206	        self.check_response(response, callback)
207	
208	        # If the response supports deferred rendering, apply template
209	        # response middleware and then render the response
210	        if hasattr(response, "render") and callable(response.render):
211	            for middleware_method in self._template_response_middleware:
```

**`django/urls/resolvers.py`** — imports(imports), calls(calls), instantiates(instantiates), __repr__(method), URLResolver(instantiates), Resolver404(instantiates), match(calls), ImproperlyConfigured(imports), ImproperlyConfigured(instantiates), Resolver404(imports), +35 more

```python
32	from .utils import get_callable
33	
34	
35	class ResolverMatch:
36	    def __init__(
37	        self,
38	        func,
39	        args,
40	        kwargs,
41	        url_name=None,
42	        app_names=None,
43	        namespaces=None,
44	        route=None,
45	        tried=None,
46	        captured_kwargs=None,
47	        extra_kwargs=None,
48	    ):
49	        self.func = func
50	        self.args = args
51	        self.kwargs = kwargs
52	        self.url_name = url_name
53	        self.route = route
54	        self.tried = tried
55	        self.captured_kwargs = captured_kwargs
56	        self.extra_kwargs = extra_kwargs
57	
58	        # If a URLRegexResolver doesn't have a namespace or app_name, it passes
59	        # in an empty value.
60	        self.app_names = [x for x in app_names if x] if app_names else []
61	        self.app_name = ":".join(self.app_names)
62	        self.namespaces = [x for x in namespaces if x] if namespaces else []
63	        self.namespace = ":".join(self.namespaces)
64	
65	        if hasattr(func, "view_class"):
66	            func = func.view_class
67	        if not hasattr(func, "__name__"):
68	            # A class-based view
69	            self._func_path = func.__class__.__module__ + "." + func.__class__.__name__
70	        else:
71	            # A function-based view
72	            self._func_path = func.__module__ + "." + func.__name__
73	

... (gap: __getitem__ (django/urls/resolvers.py:77), __repr__ (django/urls/resolvers.py:80), __reduce_ex__ (django/urls/resolvers.py:105), get_resolver (django/urls/resolvers.py:109), _get_cached_resolver (django/urls/resolvers.py:116), get_ns_resolver (django/urls/resolvers.py:121), +21 more) ...

420	        return self.language_prefix
421	
422	
423	class URLPattern:
424	    def __init__(self, pattern, callback, default_args=None, name=None):
425	        self.pattern = pattern
426	        self.callback = callback  # the view
427	        self.default_args = default_args or {}
428	        self.name = name
429	
430	    def __repr__(self):
431	        return "<%s %s>" % (self.__class__.__name__, self.pattern.describe())
432	
433	    def check(self):
434	        warnings = self._check_pattern_name()
435	        warnings.extend(self.pattern.check())
436	        warnings.extend(self._check_callback())
437	        return warnings
438	
439	    def _check_pattern_name(self):
440	        """
441	        Check that the pattern name does not contain a colon.
442	        """
443	        if self.pattern.name is not None and ":" in self.pattern.name:
444	            warning = Warning(
445	                "Your URL pattern {} has a name including a ':'. Remove the colon, to "
446	                "avoid ambiguous namespace references.".format(self.pattern.describe()),
447	                id="urls.W003",
448	            )
449	            return [warning]
450	        else:
451	            return []
452	
453	    def _check_callback(self):
454	        from django.views import View
455	
456	        view = self.callback
457	        if inspect.isclass(view) and issubclass(view, View):
458	            return [
459	                Error(
460	                    "Your URL pattern %s has an invalid view, pass %s.as_view() "
461	                    "instead of %s."
462	                    % (
463	                        self.pattern.describe(),
464	                        view.__name__,
465	                        view.__name__,
466	                    ),
467	                    id="urls.E009",
468	                )
469	            ]
470	        return []
471	
472	    def resolve(self, path):
473	        match = self.pattern.match(path)
474	        if match:
475	            new_path, args, captured_kwargs = match
476	            # Pass any default args as **kwargs.
477	            kwargs = {**captured_kwargs, **self.default_args}
478	            return ResolverMatch(
479	                self.callback,
480	                args,
481	                kwargs,
482	                self.pattern.name,
483	                route=str(self.pattern),
484	                captured_kwargs=captured_kwargs,
485	                extra_kwargs=self.default_args,
486	            )
487	
488	    @cached_property
489	    def lookup_str(self):
490	        """
491	        A string that identifies the view (e.g. 'path.to.view_function' or
492	        'path.to.ClassBasedView').
493	        """
494	        callback = self.callback
495	        if isinstance(callback, functools.partial):
496	            callback = callback.func
497	        if hasattr(callback, "view_class"):
498	            callback = callback.view_class
499	        try:
500	            return qualname(callback)
501	        except ValueError:
502	            return callback.__module__ + "." + callback.__class__.__name__
503	
504	
505	class URLResolver:
506	    def __init__(
507	        self, pattern, urlconf_name, default_kwargs=None, app_name=None, namespace=None
508	    ):
509	        self.pattern = pattern
510	        # urlconf_name is the dotted Python path to the module defining
511	        # urlpatterns. It may also be an object with an urlpatterns attribute
512	        # or urlpatterns itself.
513	        self.urlconf_name = urlconf_name
514	        self.callback = None
515	        self.default_kwargs = default_kwargs or {}
516	        self.namespace = namespace
517	        self.app_name = app_name
518	        self._reverse_dict = {}
519	        self._namespace_dict = {}
520	        self._app_dict = {}
521	        # set of dotted paths to all functions and classes that are used in
522	        # urlpatterns
523	        self._callback_strs = set()
524	        self._populated = False
525	        self._local = Local()
526	
527	    def __repr__(self):
528	        if isinstance(self.urlconf_name, list) and self.urlconf_name:
529	            # Don't bother to output the whole list, it can be huge
530	            urlconf_repr = "<%s list>" % self.urlconf_name[0].__class__.__name__
531	        else:
532	            urlconf_repr = repr(self.urlconf_name)
533	        return "<%s %s (%s:%s) %s>" % (
534	            self.__class__.__name__,
535	            urlconf_repr,
536	            self.app_name,
537	            self.namespace,
538	            self.pattern.describe(),
539	        )
540	
541	    def check(self):
542	        messages = []
543	        for pattern in self.url_patterns:
544	            messages.extend(check_resolver(pattern))
545	        return messages or self.pattern.check()
546	
547	    def _populate(self):
548	        # Short-circuit if called recursively in this thread to prevent
549	        # infinite recursion. Concurrent threads may call this at the same
550	        # time and will need to continue, so set 'populating' on a
551	        # thread-local variable.
552	        if getattr(self._local, "populating", False):
553	            return
554	        try:
555	            self._local.populating = True
556	            lookups = MultiValueDict()
557	            namespaces = {}
558	            apps = {}
559	            language_code = get_language()
560	            for url_pattern in reversed(self.url_patterns):
561	                p_pattern = url_pattern.pattern.regex.pattern
562	                p_pattern = p_pattern.removeprefix("^")
563	                if isinstance(url_pattern, URLPattern):
564	                    self._callback_strs.add(url_pattern.lookup_str)
565	                    bits = normalize(url_pattern.pattern.regex.pattern)
566	                    lookups.appendlist(
567	                        url_pattern.callback,
568	                        (
569	                            bits,
570	                            p_pattern,
571	                            url_pattern.default_args,
572	                            url_pattern.pattern.converters,
573	                        ),
574	                    )
575	                    if url_pattern.name is not None:
576	                        lookups.appendlist(
577	                            url_pattern.name,
578	                            (
579	                                bits,
580	                                p_pattern,
581	                                url_pattern.default_args,
582	                                url_pattern.pattern.converters,
583	                            ),
584	                        )
585	                else:  # url_pattern is a URLResolver.
586	                    url_pattern._populate()
587	                    if url_pattern.app_name:
588	                        apps.setdefault(url_pattern.app_name, []).append(
589	                            url_pattern.namespace
590	                        )
591	                        namespaces[url_pattern.namespace] = (p_pattern, url_pattern)
592	                    else:
593	                        for name in url_pattern.reverse_dict:
594	                            for (
595	                                matches,
596	                                pat,
597	                                defaults,
598	                                converters,
599	                            ) in url_pattern.reverse_dict.getlist(name):
600	                                new_matches = normalize(p_pattern + pat)
601	                                lookups.appendlist(
602	                                    name,
603	                                    (
604	                                        new_matches,
605	                                        p_pattern + pat,
606	                                        {**defaults, **url_pattern.default_kwargs},
607	                                        {
608	                                            **self.pattern.converters,
609	                                            **url_pattern.pattern.converters,
610	                                            **converters,
611	                                        },
612	                                    ),
613	                                )
614	                        for namespace, (
615	                            prefix,
616	                            sub_pattern,
617	                        ) in url_pattern.namespace_dict.items():
618	                            current_converters = url_pattern.pattern.converters
619	                            sub_pattern.pattern.converters.update(current_converters)
620	                            namespaces[namespace] = (p_pattern + prefix, sub_pattern)
621	                        for app_name, namespace_list in url_pattern.app_dict.items():
622	                            apps.setdefault(app_name, []).extend(namespace_list)
623	                    self._callback_strs.update(url_pattern._callback_strs)
624	            self._namespace_dict[language_code] = namespaces
625	            self._app_dict[language_code] = apps
626	            self._reverse_dict[language_code] = lookups
627	            self._populated = True
628	        finally:
629	            self._local.populating = False
630	
631	    @property
632	    def reverse_dict(self):
633	        language_code = get_language()
634	        if language_code not in self._reverse_dict:
635	            self._populate()
636	        return self._reverse_dict[language_code]
637	
638	    @property
639	    def namespace_dict(self):
640	        language_code = get_language()
641	        if language_code not in self._namespace_dict:
642	            self._populate()
643	        return self._namespace_dict[language_code]
644	
645	    @property
646	    def app_dict(self):
647	        language_code = get_language()
648	        if language_code not in self._app_dict:
649	            self._populate()
650	        return self._app_dict[language_code]
651	
652	    @staticmethod
653	    def _extend_tried(tried, pattern, sub_tried=None):
654	        if sub_tried is None:
655	            tried.append([pattern])
656	        else:
657	            tried.extend([pattern, *t] for t in sub_tried)
658	
659	    @staticmethod
660	    def _join_route(route1, route2):
661	        """Join two routes, without the starting ^ in the second route."""
662	        if not route1:
663	            return route2
664	        route2 = route2.removeprefix("^")
665	        return route1 + route2
666	
667	    def _is_callback(self, name):
668	        if not self._populated:
669	            self._populate()
670	        return name in self._callback_strs
671	
672	    def resolve(self, path):
673	        path = str(path)  # path may be a reverse_lazy object
674	        tried = []
675	        match = self.pattern.match(path)
676	        if match:
677	            new_path, args, kwargs = match
678	            for pattern in self.url_patterns:
679	                try:
680	                    sub_match = pattern.resolve(new_path)
681	                except Resolver404 as e:
682	                    self._extend_tried(tried, pattern, e.args[0].get("tried"))
683	                else:
684	                    if sub_match:
685	                        # Merge captured arguments in match with submatch
686	                        sub_match_dict = {**kwargs, **self.default_kwargs}
687	                        # Update the sub_match_dict with the kwargs from the
688	                        # sub_match.
689	                        sub_match_dict.update(sub_match.kwargs)
690	                        # If there are *any* named groups, ignore all non-named
691	                        # groups. Otherwise, pass all non-named arguments as
692	                        # positional arguments.
693	                        sub_match_args = sub_match.args
694	                        if not sub_match_dict:
695	                            sub_match_args = args + sub_match.args
696	                        current_route = (
697	                            ""
698	                            if isinstance(pattern, URLPattern)
699	                            else str(pattern.pattern)
700	                        )
701	                        self._extend_tried(tried, pattern, sub_match.tried)
702	                        return ResolverMatch(
703	                            sub_match.func,
704	                            sub_match_args,
705	                            sub_match_dict,
706	                            sub_match.url_name,
707	                            [self.app_name, *sub_match.app_names],
708	                            [self.namespace, *sub_match.namespaces],
709	                            self._join_route(current_route, sub_match.route),
710	                            tried,
711	                            captured_kwargs=sub_match.captured_kwargs,
712	                            extra_kwargs={
713	                                **self.default_kwargs,
714	                                **sub_match.extra_kwargs,
715	                            },
716	                        )
717	                    tried.append([pattern])
718	            raise Resolver404({"tried": tried, "path": new_path})
719	        raise Resolver404({"path": path})
720	
721	    @cached_property
722	    def urlconf_module(self):
723	        if isinstance(self.urlconf_name, str):
724	            return import_module(self.urlconf_name)
725	        else:
726	            return self.urlconf_name
727	
728	    @cached_property
729	    def url_patterns(self):
730	        # urlconf_module might be a valid set of patterns, so we default to it
731	        patterns = getattr(self.urlconf_module, "urlpatterns", self.urlconf_module)
732	        try:
733	            iter(patterns)
734	        except TypeError as e:
735	            msg = (
736	                "The included URLconf '{name}' does not appear to have "
737	                "any patterns in it. If you see the 'urlpatterns' variable "
738	                "with valid patterns in the file then the issue is probably "
739	                "caused by a circular import."
740	            )
741	            raise ImproperlyConfigured(msg.format(name=self.urlconf_name)) from e
742	        return patterns
743	
744	    def resolve_error_handler(self, view_type):
745	        callback = getattr(self.urlconf_module, "handler%s" % view_type, None)
746	        if not callback:
747	            # No handler specified in file; use lazy import, since
748	            # django.conf.urls imports this file.
749	            from django.conf import urls
750	
751	            callback = getattr(urls, "handler%s" % view_type)
752	        return get_callable(callback)
753	
754	    def reverse(self, lookup_view, *args, **kwargs):
755	        return self._reverse_with_prefix(lookup_view, "", *args, **kwargs)
756	
757	    def _reverse_with_prefix(self, lookup_view, _prefix, *args, **kwargs):
758	        if args and kwargs:
```

**`django/urls/exceptions.py`** — Resolver404(class)

```python
1	from django.http import Http404
2	
3	
4	class Resolver404(Http404):
5	    pass
6	
7	
8	class NoReverseMatch(Exception):
9	    pass
```

**Not shown above — explore these names for their source**

- django/core/handlers/wsgi.py: WSGIHandler:113, __init__:116, __call__:120
- django/core/handlers/exception.py: convert_exception_to_response:25, inner:41, inner:54
- django/contrib/gis/geos/mutable_list.py: insert:196, append:188, ListMixin:16
- tests/urlpatterns/tests.py: test_path_trailing_newlines:248, test_nonmatching_urls:296, test_resolve_value_error_means_no_match:412, tests.py:1
- django/contrib/admin/views/autocomplete.py: get:14, process_request:67
- django/shortcuts.py: get_object_or_404:79, aget_object_or_404:110, get_list_or_404:129, aget_list_or_404:154
- ... and 29 more files

---
> **Complete source for 3 files is included above — do NOT re-read them.** If your question also needs files/symbols listed under "Not shown above" (or any area this call didn't cover), make ANOTHER codegraph_explore targeting those names — it returns the same source with line numbers and is cheaper and more complete than reading. Reserve Read for a single specific line range explore can't surface.
````

## 判定

- 字符预算：通过
- 事实覆盖率：通过
- 重复结果缩减：通过
- 综合 Gate：PASS

> 本文件保存的是固定测试时的原始结果。重新运行后应导出到另一个目录，以便进行版本间对比。
