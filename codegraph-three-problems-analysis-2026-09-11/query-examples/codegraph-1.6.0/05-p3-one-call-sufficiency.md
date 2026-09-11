# 一次查询充分后是否停止

- 用例 ID：`p3-one-call-sufficiency`
- 对应问题：P3
- CodeGraph：1.6.0
- Django 项目：`D:\工作空间\harbor\exp\django`
- 记录时间：2026-09-10T12:25:23.857Z
- 用例配置哈希：`早期记录未提供`

## 测试目的

The primary query names enough endpoints for one explore response to answer the task.

## 给 Agent 的任务

```text
Describe how BaseHandler._get_response reaches the resolved callback, including URLResolver.resolve and ResolverMatch. Cite the relevant files and do not discuss unrelated request features.
```

## 直接 MCP 查询

### 第 1 次调用

```text
BaseHandler._get_response resolve_request URLResolver.resolve ResolverMatch callback
```

复现命令：

```powershell
codegraph explore "BaseHandler._get_response resolve_request URLResolver.resolve ResolverMatch callback" --path "D:\工作空间\harbor\exp\django"
```

| 指标 | 结果 |
|---|---:|
| 响应字符数 | 20078 |
| 估算 token | 6693 |
| 查询耗时 | 558.3 ms |
| 事实覆盖率 | 1 |
| 概念命中 | 5/5 |
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
**Exploration: BaseHandler._get_response resolve_request URLResolver.resolve ResolverMatch callback**

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
- ... and 6 more

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
- ... and 59 more

**Source Code**

> The code below is the **verbatim, current on-disk source** of these files — re-read from disk on this call and line-numbered, byte-for-byte identical to what the Read tool returns. It is NOT a summary, outline, or stale cache. Treat each block as a Read you have already performed: do not Read a file shown here.

**`django/urls/resolvers.py`** — calls(calls), Resolver404(instantiates), ResolverMatch(instantiates), __init__(method), __repr__(method), instantiates(instantiates), match(calls), decorates(decorates), imports(imports), ImproperlyConfigured(imports), +34 more

```python
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

... (gap) ...

596	                                pat,
597	                                defaults,
598	                                converters,
599	                            ) in url_pattern.reverse_dict.getlist(name):
600	                                new_matches = normalize(p_pattern + pat)
601	                                lookups.appendlist(
602	                                    name,
603	                                    (
604	                                        new_matches,

... (gap) ...

614	                        for namespace, (
615	                            prefix,
616	                            sub_pattern,
617	                        ) in url_pattern.namespace_dict.items():
618	                            current_converters = url_pattern.pattern.converters
619	                            sub_pattern.pattern.converters.update(current_converters)
620	                            namespaces[namespace] = (p_pattern + prefix, sub_pattern)
621	                        for app_name, namespace_list in url_pattern.app_dict.items():
622	                            apps.setdefault(app_name, []).extend(namespace_list)

... (gap) ...

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
```

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

- tests/urlpatterns/tests.py: test_path_trailing_newlines:248, test_nonmatching_urls:296, test_resolve_value_error_means_no_match:412, tests.py:1
- django/contrib/admin/views/autocomplete.py: get:14, process_request:67
- django/shortcuts.py: get_object_or_404:79, aget_object_or_404:110, get_list_or_404:129, aget_list_or_404:154
- django/core/handlers/asgi.py: ASGIHandler:150, __init__:157
- django/core/handlers/wsgi.py: WSGIHandler:113, __init__:116
- django/urls/conf.py: _path:62, conf.py:1
- django/views/debug.py: technical_404_response:609, debug.py:1
- django/urls/base.py: set_urlconf:148, base.py:1
- django/test/client.py: ClientHandler:158, AsyncClientHandler:213, client.py:1
- django/http/response.py: Http404:740
- ... and 24 more files

---
> **Complete source for 3 files is included above — do NOT re-read them.** If your question also needs files/symbols listed under "Not shown above" (or any area this call didn't cover), make ANOTHER codegraph_explore targeting those names — it returns the same source with line numbers and is cheaper and more complete than reading. Reserve Read for a single specific line range explore can't surface.

> **Exploration guidance — advisory only, NOT a quota: this project (~3,019 files indexed) is usually covered in ≈2 focused explore calls, and extra calls are never rejected or rate-limited.** If the response above does not fully cover your question, run another codegraph_explore on the uncovered symbols — it is cheaper and more complete than Read. Only stop exploring when the response actually covers the flow you asked about.
````

## 判定

- 字符预算：通过
- 事实覆盖率：通过
- 重复结果缩减：通过
- 综合 Gate：PASS

> 本文件保存的是固定测试时的原始结果。重新运行后应导出到另一个目录，以便进行版本间对比。
