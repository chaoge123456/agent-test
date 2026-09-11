# 噪声与拼写错误：get_respnse 查询

- 用例 ID：`p2-noisy-typo-query`
- 对应问题：P2
- CodeGraph：1.6.0
- Django 项目：`D:\工作空间\harbor\exp\django`
- 记录时间：2026-09-10T12:25:23.857Z
- 用例配置哈希：`早期记录未提供`

## 测试目的

Real agent queries contain prose, partial names, and occasional typos.

## 给 Agent 的任务

```text
Find the request-response path around get_respnse (the spelling may be wrong), including middleware and URL matching, and explain the corrected symbol chain with file locations.
```

## 直接 MCP 查询

### 第 1 次调用

```text
request response path get_respnse middleware URL matching callback
```

复现命令：

```powershell
codegraph explore "request response path get_respnse middleware URL matching callback" --path "D:\工作空间\harbor\exp\django"
```

| 指标 | 结果 |
|---|---:|
| 响应字符数 | 24090 |
| 估算 token | 8030 |
| 查询耗时 | 616.2 ms |
| 事实覆盖率 | 0.2 |
| 概念命中 | 1/4 |
| 文件命中 | 0/1 |
| MCP 错误 | 否 |

#### 完整原始返回

````text
**Exploration: request response path get_respnse middleware URL matching callback**

Found 69 symbols across 5 files.

**Blast radius — what depends on these (update/verify before editing)**

- `callback` (django/utils/decorators.py:156) — 1 caller in `django/utils/decorators.py`; no tests found within 3 caller hops
- `match` (django/http/request.py:775) — 14 callers in `django/contrib/gis/db/models/lookups.py`, `django/core/validators.py`, `django/db/backends/oracle/operations.py`, `django/db/models/functions/comparison.py` +4 more; tests: `django/test/client.py`, `tests/test_client_regress/views.py`
- `match` (django/urls/resolvers.py:203) — 15 callers in `django/contrib/admin/options.py`, `django/contrib/gis/db/backends/oracle/operations.py`, `django/contrib/gis/db/models/fields.py`, `django/contrib/gis/forms/widgets.py` +7 more; tests: `tests/gis_tests/geo3d/tests.py`, `tests/gis_tests/geoapp/test_functions.py`, `tests/utils_tests/test_regex_helper.py`
- `match` (django/urls/resolvers.py:325) — 3 callers; tests: `tests/urlpatterns/test_resolvers.py`
- `match` (django/urls/resolvers.py:407) — 3 callers in `django/core/management/commands/makemessages.py`, `django/urls/resolvers.py`, `django/utils/translation/trans_real.py`; tested via callers: `tests/urlpatterns/test_resolvers.py`, `tests/urlpatterns_reverse/tests.py` +1

**Relationships**

**calls:**
- callback → process_response
- _post_process_request → process_template_response
- _post_process_request → add_post_render_callback
- _post_process_request → process_response
- _view_wrapper → _post_process_request
- _view_wrapper → _post_process_request
- process_response → flatpage
- process_template_response → append
- add_post_render_callback → append
- process_response → add_post_render_callback
- ... and 126 more

**references:**
- _post_process_request → callback
- _decorator → _view_wrapper
- _make_decorator → _decorator
- MiddlewareMixinTests → FlatpageFallbackMiddleware
- flatpage → FlatPage
- match → MediaType
- path → RoutePattern
- path → _path
- is_language_prefix_patterns_used → LocalePrefixPattern
- include → LocalePrefixPattern
- ... and 3 more

**extends:**
- FlatpageFallbackMiddleware → MiddlewareMixin
- RegexPattern → CheckURLMixin
- RoutePattern → CheckURLMixin

**instantiates:**
- flatpage → HttpResponsePermanentRedirect
- match → MediaType
- accepted_types → MediaType
- accepted_type → MediaType
- test_empty → MediaType
- RegexPattern → LocaleRegexDescriptor
- _get_cached_resolver → RegexPattern
- RoutePattern → LocaleRegexRouteDescriptor
- test_str → RoutePattern
- test_has_converters → RoutePattern
- ... and 21 more

**decorates:**
- path → cached_property
- default_auto_field → cached_property
- is_collapsible → cached_property
- is_collapsible → cached_property
- username_is_unique → cached_property
- natural_key_defined → cached_property
- DEFAULT_PASSWORD_LIST_PATH → cached_property
- ct_field_attname → cached_property
- cache_name → cached_property
- related_manager_cls → cached_property
- ... and 4 more

**Source Code**

> The code below is the **verbatim, current on-disk source** of these files — re-read from disk on this call and line-numbered, byte-for-byte identical to what the Read tool returns. It is NOT a summary, outline, or stale cache. Treat each block as a Read you have already performed: do not Read a file shown here.

**`django/contrib/staticfiles/storage.py`** — calls(calls), hashed_name(method), instantiates(instantiates), join(calls), path(method)

```python
52	            self.base_location = None
53	            self.location = None
54	
55	    def path(self, name):
56	        if not self.location:
57	            raise ImproperlyConfigured(
58	                "You're using the staticfiles app "
59	                "without having set the STATIC_ROOT "
60	                "setting to a filesystem path."
61	            )
62	        return super().path(name)
63	
64	
65	class HashedFilesMixin:

... (gap: __init__ (django/contrib/staticfiles/storage.py:133), file_hash (django/contrib/staticfiles/storage.py:155)) ...

163	            hasher.update(chunk)
164	        return hasher.hexdigest()[:12]
165	
166	    def hashed_name(self, name, content=None, filename=None):
167	        # `filename` is the name of file to hash if `content` isn't given.
168	        # `name` is the base name to construct the new hashed filename from.
169	        parsed_name = urlsplit(unquote(name))
170	        clean_name = parsed_name.path.strip()
171	        filename = (filename and urlsplit(unquote(filename)).path.strip()) or clean_name
172	        opened = content is None
173	        if opened:
174	            if not self.exists(filename):
175	                raise ValueError(
176	                    "The file '%s' could not be found with %r." % (filename, self)
177	                )
178	            try:
179	                content = self.open(filename)
180	            except OSError:
181	                # Handle directory paths and fragments
182	                return name
183	        try:
184	            file_hash = self.file_hash(clean_name, content)
185	        finally:
186	            if opened:
187	                content.close()
188	        path, filename = os.path.split(clean_name)
189	        root, ext = os.path.splitext(filename)
190	        file_hash = (".%s" % file_hash) if file_hash else ""
191	        hashed_name = os.path.join(path, "%s%s%s" % (root, file_hash, ext))
192	        unparsed_name = list(parsed_name)
193	        unparsed_name[2] = hashed_name
194	        # Special casing for a @font-face hack, like url(myfont.eot?#iefix")
195	        # http://www.fontspring.com/blog/the-new-bulletproof-font-face-syntax
196	        if "?#" in name and not unparsed_name[3]:
197	            unparsed_name[2] += "?"
198	        return urlunsplit(unparsed_name)
199	
200	    def _url(self, hashed_name_func, name, force=False, hashed_files=None):
201	        """
```

**`django/utils/decorators.py`** — _make_decorator(function), _decorator(function), _decorator(references), calls(calls), _post_process_request(calls), _pre_process_request(calls), _process_exception(calls), _view_wrapper(function), process_response(calls), _post_process_request(function), +7 more

```python
127	
128	            def _pre_process_request(request, *args, **kwargs):
129	                if hasattr(middleware, "process_request"):
130	                    result = middleware.process_request(request)
131	                    if result is not None:
132	                        return result
133	                if hasattr(middleware, "process_view"):
134	                    result = middleware.process_view(request, view_func, args, kwargs)
135	                    if result is not None:
136	                        return result
137	                return None
138	
139	            def _process_exception(request, exception):
140	                if hasattr(middleware, "process_exception"):
141	                    result = middleware.process_exception(request, exception)
142	                    if result is not None:
143	                        return result
144	                raise
145	
146	            def _post_process_request(request, response):
147	                if hasattr(response, "render") and callable(response.render):
148	                    if hasattr(middleware, "process_template_response"):
149	                        response = middleware.process_template_response(
150	                            request, response
151	                        )
152	                    # Defer running of process_response until after the
153	                    # template has been rendered:
154	                    if hasattr(middleware, "process_response"):
155	
156	                        def callback(response):
157	                            return middleware.process_response(request, response)
158	
159	                        response.add_post_render_callback(callback)
160	                else:
161	                    if hasattr(middleware, "process_response"):
162	                        return middleware.process_response(request, response)
163	                return response
164	
165	            if iscoroutinefunction(view_func):
166	
167	                async def _view_wrapper(request, *args, **kwargs):
168	                    result = _pre_process_request(request, *args, **kwargs)
169	                    if result is not None:
170	                        return result
171	
172	                    try:
173	                        response = await view_func(request, *args, **kwargs)
174	                    except Exception as e:
175	                        result = _process_exception(request, e)
176	                        if result is not None:
177	                            return result
178	
179	                    return _post_process_request(request, response)
180	
181	            else:
182	
183	                def _view_wrapper(request, *args, **kwargs):
```

**`django/urls/resolvers.py`** — describe(calls), instantiates(instantiates), CheckURLMixin(extends), describe(method), _check_pattern_startswith_slash(calls), _route_to_regex(calls), _get_cached_resolver(function), RegexPattern(class), RegexPattern(instantiates), LocaleRegexDescriptor(class), +25 more

```python
177	        if self._regex.startswith(("/", "^/", "^\\/")) and not self._regex.endswith(
178	            "/"
179	        ):
180	            warning = Warning(
181	                "Your URL pattern {} has a route beginning with a '/'. Remove this "
182	                "slash as it is unnecessary. If this pattern is targeted in an "
183	                "include(), ensure the include() pattern has a trailing '/'.".format(
184	                    self.describe()
185	                ),
186	                id="urls.W002",
187	            )
188	            return [warning]
189	        else:
190	            return []
191	
192	
193	class RegexPattern(CheckURLMixin):
194	    regex = LocaleRegexDescriptor()
195	
196	    def __init__(self, regex, name=None, is_endpoint=False):
197	        self._regex = regex
198	        self._regex_dict = {}
199	        self._is_endpoint = is_endpoint
200	        self.name = name
201	        self.converters = {}
202	
203	    def match(self, path):
204	        match = (
205	            self.regex.fullmatch(path)
206	            if self._is_endpoint and self.regex.pattern.endswith("$")
207	            else self.regex.search(path)
208	        )
209	        if match:
210	            # If there are any named groups, use those as kwargs, ignoring
211	            # non-named groups. Otherwise, pass all non-named arguments as
212	            # positional arguments.
213	            kwargs = match.groupdict()
214	            args = () if kwargs else match.groups()
215	            kwargs = {k: v for k, v in kwargs.items() if v is not None}
216	            return path[match.end() :], args, kwargs
217	        return None
218	
219	    def check(self):
220	        warnings = []
221	        warnings.extend(self._check_pattern_startswith_slash())
222	        if not self._is_endpoint:
223	            warnings.extend(self._check_include_trailing_dollar())
224	        return warnings
225	
226	    def _check_include_trailing_dollar(self):
227	        if self._regex.endswith("$") and not self._regex.endswith(r"\$"):
228	            return [
229	                Warning(
230	                    "Your URL pattern {} uses include with a route ending with a '$'. "
231	                    "Remove the dollar from the route to avoid problems including "
232	                    "URLs.".format(self.describe()),
233	                    id="urls.W001",
234	                )
235	            ]
236	        else:
237	            return []
238	
239	    def __str__(self):
240	        return str(self._regex)
241	
242	
243	_PATH_PARAMETER_COMPONENT_RE = _lazy_re_compile(

... (gap: whitespace_set (django/urls/resolvers.py:247), _route_to_regex (django/urls/resolvers.py:251)) ...

256	    and {'pk': <django.urls.converters.IntConverter>}.
257	    """
258	    parts = ["^"]
259	    all_converters = get_converters()
260	    converters = {}
261	    previous_end = 0
262	    for match_ in _PATH_PARAMETER_COMPONENT_RE.finditer(route):
263	        if not whitespace_set.isdisjoint(match_[0]):
264	            raise ImproperlyConfigured(
265	                f"URL route {route!r} cannot contain whitespace in angle brackets <…>."
266	            )
267	        # Default to make converter "str" if unspecified (parameter always
268	        # matches something).

... (gap: LocaleRegexRouteDescriptor (django/urls/resolvers.py:294), __get__ (django/urls/resolvers.py:295), RoutePattern (django/urls/resolvers.py:315), __init__ (django/urls/resolvers.py:318)) ...

322	        self._is_endpoint = is_endpoint
323	        self.name = name
324	
325	    def match(self, path):
326	        # Only use regex overhead if there are converters.
327	        if self.converters:
328	            if match := self.regex.search(path):
329	                # RoutePattern doesn't allow non-named groups so args are
330	                # ignored.
331	                kwargs = match.groupdict()
332	                for key, value in kwargs.items():
333	                    converter = self.converters[key]
334	                    try:
335	                        kwargs[key] = converter.to_python(value)
336	                    except ValueError:
337	                        return None
338	                return path[match.end() :], (), kwargs
339	        # If this is an endpoint, the path should be exactly the same as the
340	        # route.
341	        elif self._is_endpoint:
342	            if self._route == path:
343	                return "", (), {}
344	        # If this isn't an endpoint, the path should start with the route.

... (gap: check (django/urls/resolvers.py:349), _check_pattern_unmatched_angle_brackets (django/urls/resolvers.py:366), __str__ (django/urls/resolvers.py:385), LocalePrefixPattern (django/urls/resolvers.py:389)) ...

390	    def __init__(self, prefix_default_language=True):
391	        self.prefix_default_language = prefix_default_language
392	        self.converters = {}
393	
394	    @property
395	    def regex(self):
396	        # This is only used by reverse() and cached in _reverse_dict.
397	        return re.compile(re.escape(self.language_prefix))
398	
399	    @property
400	    def language_prefix(self):
401	        language_code = get_language() or settings.LANGUAGE_CODE
402	        if language_code == settings.LANGUAGE_CODE and not self.prefix_default_language:
403	            return ""
404	        else:
405	            return "%s/" % language_code
406	
407	    def match(self, path):
408	        language_prefix = self.language_prefix
409	        if path.startswith(language_prefix):
410	            return path.removeprefix(language_prefix), (), {}
411	        return None
412	
413	    def check(self):
414	        return []
415	
416	    def describe(self):
417	        return "'{}'".format(self)
418	
419	    def __str__(self):
420	        return self.language_prefix
421	
422	
423	class URLPattern:
```

**`django/core/management/commands/makemessages.py`** — extends(extends), handle(method), handle_extensions(calls), write(calls), get_text_list(calls), append(calls), filter(calls), match(calls), build_potfiles(calls), lower(calls), +18 more

```python
32	
33	def check_programs(*programs):
34	    for program in programs:
35	        if find_command(program) is None:
36	            raise CommandError(
37	                f"Can't find {program}. Make sure you have GNU gettext tools "
38	                "0.19 or newer installed."
39	            )
40	
41	
42	def is_valid_locale(locale):
43	    return re.match(r"^[a-z]+$", locale) or re.match(r"^[a-z]+_[A-Z0-9].*$", locale)
44	
45	
46	@total_ordering
47	class TranslatableFile:
48	    def __init__(self, dirpath, file_name, locale_dir):
49	        self.file = file_name
50	        self.dirpath = dirpath
51	        self.locale_dir = locale_dir
52	
53	    def __repr__(self):
54	        return "<%s: %s>" % (
55	            self.__class__.__name__,
56	            os.sep.join([self.dirpath, self.file]),
57	        )
58	
59	    def __eq__(self, other):
60	        return self.path == other.path
61	
62	    def __lt__(self, other):
63	        return self.path < other.path
64	
65	    @property
66	    def path(self):
67	        return os.path.join(self.dirpath, self.file)
68	
69	
70	class BuildFile:
71	    """
72	    Represent the state of a translatable file during the build process.
73	    """
74	
75	    def __init__(self, command, domain, translatable):
76	        self.command = command
77	        self.domain = domain
78	        self.translatable = translatable
79	
80	    @cached_property
81	    def is_templatized(self):
82	        if self.domain == "django":
83	            file_ext = os.path.splitext(self.translatable.file)[1]
84	            return file_ext != ".py"
85	        return False
86	
87	    @cached_property
88	    def path(self):
89	        return self.translatable.path
90	
91	    @cached_property
92	    def work_path(self):
93	        """
94	        Path to a file which is being fed into GNU gettext pipeline. This may
95	        be either a translatable or its preprocessed version.
96	        """
97	        if not self.is_templatized:
98	            return self.path
99	        filename = f"{self.translatable.file}.py"
100	        return os.path.join(self.translatable.dirpath, filename)
101	
102	    def preprocess(self):
103	        """
104	        Preprocess (if necessary) a translatable file before passing it to
105	        xgettext GNU gettext utility.
106	        """
107	        if not self.is_templatized:
108	            return
109	
110	        with open(self.path, encoding="utf-8") as fp:
111	            src_data = fp.read()
112	
113	        if self.domain == "django":
114	            content = templatize(src_data, origin=self.path[2:])
115	
116	        with open(self.work_path, "w", encoding="utf-8") as fp:
117	            fp.write(content)
118	
119	    def postprocess_messages(self, msgs):
120	        """
121	        Postprocess messages generated by xgettext GNU gettext utility.
122	
123	        Transform paths as if these messages were generated from original
124	        translatable files rather than from preprocessed versions.
125	        """
126	        if not self.is_templatized:
127	            return msgs
128	
129	        # Remove '.py' suffix
130	        if os.name == "nt":
131	            # Preserve '.\' prefix on Windows to respect gettext behavior
132	            old_path = self.work_path
133	            new_path = self.path
134	        else:
135	            old_path = self.work_path[2:]
136	            new_path = self.path[2:]
137	
138	        return re.sub(
139	            r"^(#: .*)(" + re.escape(old_path) + r")",
140	            lambda match: match[0].replace(old_path, new_path),
141	            msgs,
142	            flags=re.MULTILINE,
143	        )
144	
145	    def cleanup(self):
146	        """
147	        Remove a preprocessed copy of a translatable file (if any).
148	        """
149	        if self.is_templatized:
```

**`django/http/request.py`** — calls(calls), cached_property(decorates), accepted_types(method), MediaType(class), MediaType(instantiates), MediaType(references), accepted_type(method), match(calls), match(method), specificity(method), +6 more

```python
749	
750	
751	class MediaType:
752	    def __init__(self, media_type_raw_line):
753	        full_type, self.params = parse_header_parameters(
754	            media_type_raw_line if media_type_raw_line else ""
755	        )
756	        self.main_type, _, self.sub_type = full_type.partition("/")
757	
758	    def __str__(self):
759	        params_str = "".join("; %s=%s" % (k, v) for k, v in self.params.items())
760	        return "%s%s%s" % (
761	            self.main_type,
762	            ("/%s" % self.sub_type) if self.sub_type else "",
763	            params_str,
764	        )
765	
766	    def __repr__(self):
767	        return "<%s: %s>" % (self.__class__.__qualname__, self)
768	
769	    @cached_property
770	    def range_params(self):
771	        params = self.params.copy()
772	        params.pop("q", None)
773	        return params
774	
775	    def match(self, other):
776	        if not other:
777	            return False
778	
779	        if not isinstance(other, MediaType):
780	            other = MediaType(other)
781	
782	        main_types = [self.main_type, other.main_type]
783	        sub_types = [self.sub_type, other.sub_type]
784	
785	        # Main types and sub types must be defined.
786	        if not all((*main_types, *sub_types)):
787	            return False
788	
789	        # Main types must match or one be "*", same for sub types.
790	        for this_type, other_type in (main_types, sub_types):
791	            if this_type != other_type and this_type != "*" and other_type != "*":
792	                return False
793	
794	        if bool(self.range_params) == bool(other.range_params):
795	            # If both have params or neither have params, they must be
796	            # identical.
797	            result = self.range_params == other.range_params
798	        else:
799	            # If self has params and other does not, it's a match.
800	            # If other has params and self does not, don't match.
801	            result = bool(self.range_params or not other.range_params)
802	        return result
803	
804	    @cached_property
805	    def quality(self):
806	        try:
807	            quality = float(self.params.get("q", 1))
808	        except ValueError:
809	            # Discard invalid values.
810	            return 1
```

**Not shown above — explore these names for their source**

- django/db/migrations/writer.py: path:289, MigrationWriter:118, __init__:124, as_string:129, basedir:219, filename:285, +3 more
- django/utils/http.py: parse_etags:226, quote_etag:243, parse_http_date:100
- django/utils/functional.py: cached_property:7, func:19, __init__:25, __set_name__:29, __get__:39
- django/core/validators.py: __call__:244, validate_domain_part:260
- tests/urlpatterns/test_resolvers.py: test_match_lazy_route_without_converters:24, test_match_lazy_route_endpoint:29, test_match_lazy_route_with_converters:34, test_str:15, test_has_converters:18, RoutePatternTests:14
- django/contrib/gis/gdal/raster/source.py: _preprocess_input:235, __init__:85, clone:498
- django/test/client.py: _encode_data:441, _encode_json:453, _parse_json:956
- django/contrib/flatpages/middleware.py: process_response:8, FlatpageFallbackMiddleware:7
- django/urls/converters.py: to_python:8, IntConverter:5
- django/db/models/functions/json.py: join:85, JSONObject:68
- ... and 62 more files

---
> **Complete source for 5 files is included above — do NOT re-read them.** If your question also needs files/symbols listed under "Not shown above" (or any area this call didn't cover), make ANOTHER codegraph_explore targeting those names — it returns the same source with line numbers and is cheaper and more complete than reading. Reserve Read for a single specific line range explore can't surface.

> **Exploration guidance — advisory only, NOT a quota: this project (~3,019 files indexed) is usually covered in ≈2 focused explore calls, and extra calls are never rejected or rate-limited.** If the response above does not fully cover your question, run another codegraph_explore on the uncovered symbols — it is cheaper and more complete than Read. Only stop exploring when the response actually covers the flow you asked about.
````

## 判定

- 字符预算：通过
- 事实覆盖率：失败
- 重复结果缩减：通过
- 综合 Gate：FAIL

> 本文件保存的是固定测试时的原始结果。重新运行后应导出到另一个目录，以便进行版本间对比。
