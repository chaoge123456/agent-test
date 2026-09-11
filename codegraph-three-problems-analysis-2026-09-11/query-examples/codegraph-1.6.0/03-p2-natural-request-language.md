# 自然语言查询：从请求到 view

- 用例 ID：`p2-natural-request-language`
- 对应问题：P2
- CodeGraph：1.6.0
- Django 项目：`D:\工作空间\harbor\exp\django`
- 记录时间：2026-09-10T12:25:23.857Z
- 用例配置哈希：`早期记录未提供`

## 测试目的

An agent often asks in prose before it knows exact symbol names.

## 给 Agent 的任务

```text
How does Django take an incoming request, resolve its URL, and call the selected Python view? Give the concrete control-flow chain and source files.
```

## 直接 MCP 查询

### 第 1 次调用

```text
how Django takes an incoming request resolves its URL and calls the selected Python view
```

复现命令：

```powershell
codegraph explore "how Django takes an incoming request resolves its URL and calls the selected Python view" --path "D:\工作空间\harbor\exp\django"
```

| 指标 | 结果 |
|---|---:|
| 响应字符数 | 24996 |
| 估算 token | 8332 |
| 查询耗时 | 699.9 ms |
| 事实覆盖率 | 0.429 |
| 概念命中 | 3/5 |
| 文件命中 | 0/2 |
| MCP 错误 | 否 |

#### 完整原始返回

````text
**Exploration: how Django takes an incoming request resolves its URL and calls the selected Python view**

Found 73 symbols across 8 files.

**Blast radius — what depends on these (update/verify before editing)**

- `View` (django/views/generic/base.py:37) — 32 callers in `django/views/generic/dates.py`, `django/views/generic/detail.py`, `django/views/generic/edit.py`, `django/views/generic/list.py` +4 more; tests: `tests/admin_docs/views.py`, `tests/async/tests.py`, `tests/auth_tests/test_mixins.py`, `tests/auth_tests/urls.py` +13
- `Select` (django/forms/widgets.py:869) — 21 callers in `django/contrib/admin/widgets.py`, `django/forms/fields.py`, `django/forms/widgets.py`; tests: `tests/forms_tests/field_tests/test_base.py`, `tests/forms_tests/tests/test_forms.py`, `tests/forms_tests/tests/test_i18n.py`, `tests/forms_tests/widget_tests/test_select.py` +1
- `view` (django/views/generic/base.py:97) — 1 caller in `django/views/generic/base.py`; tested via callers: `tests/admin_views/test_autocomplete_view.py`, `tests/async/tests.py` +22
- `python` (django/core/management/commands/shell.py:68) — 1 caller; tests: `tests/shell/tests.py`
- `select` (django/contrib/gis/db/backends/mysql/operations.py:27) — 1 caller in `django/core/management/commands/shell.py`; no tests found within 3 caller hops

**Relationships**

**extends:**
- Select → ChoiceWidget
- ChoiceWidget → Widget
- TemplateView → View
- RedirectView → View
- BaseDateListView → View
- BaseDetailView → View
- ProcessFormView → View
- BaseListView → View
- JavaScriptCatalog → View
- XViewClass → View
- ... and 16 more

**instantiates:**
- CustomChoiceField → Select
- FrameworkForm → Select
- FrameworkForm → Select
- CopyForm → Select
- test_constructor_attrs → Select
- test_choices_constructor → Select
- test_choices_constructor_generator → Select
- python → OrderedSet
- handle → CommandError
- file_hash → MD5

**references:**
- __init__ → Select
- ChoiceField → Select
- SelectDateWidget → Select
- SelectTest → Select
- get_formset → Select
- test_raw_id_fields_widget_override → Select
- test_default_foreign_key_widget → Select
- as_view → view
- MySQLOperations → WKTAdapter
- DatabaseWrapper → MySQLOperations
- ... and 10 more

**calls:**
- use_required_attribute → _choice_has_empty_value
- view → setup
- view → dispatch
- as_view → update
- get_urls → as_view
- password_change → as_view
- password_change_done → as_view
- logout → as_view
- login → as_view
- autocomplete_view → as_view
- ... and 60 more

**decorates:**
- select → cached_property

**Source Code**

> The code below is the **verbatim, current on-disk source** of these files — re-read from disk on this call and line-numbered, byte-for-byte identical to what the Read tool returns. It is NOT a summary, outline, or stale cache. Treat each block as a Read you have already performed: do not Read a file shown here.

**`django/views/generic/base.py`** — View(class), TemplateView(class), extends(extends), RedirectView(class), calls(calls), instantiates(instantiates), _allowed_methods(calls), decorates(decorates), __init__(method), _allowed_methods(method), +11 more

```python
51	        "trace",
52	    ]
53	
54	    def __init__(self, **kwargs):
55	        """
56	        Constructor. Called in the URLconf; can contain helpful extra
57	        keyword arguments, and other things.
58	        """
59	        # Go through keyword arguments, and either save their values to our
60	        # instance, or raise an error.
61	        for key, value in kwargs.items():
62	            setattr(self, key, value)
63	
64	    @classproperty
65	    def view_is_async(cls):
66	        handlers = [
67	            getattr(cls, method)
68	            for method in cls.http_method_names
69	            if (method != "options" and hasattr(cls, method))
70	        ]
71	        if not handlers:
72	            return False
73	        is_async = iscoroutinefunction(handlers[0])
74	        if not all(iscoroutinefunction(h) == is_async for h in handlers[1:]):
75	            raise ImproperlyConfigured(
76	                f"{cls.__qualname__} HTTP handlers must either be all sync or all "
77	                "async."
78	            )
79	        return is_async
80	
81	    @classonlymethod
82	    def as_view(cls, **initkwargs):
83	        """Main entry point for a request-response process."""
84	        for key in initkwargs:

... (gap) ...

94	                    "attributes of the class." % (cls.__name__, key)
95	                )
96	
97	        def view(request, *args, **kwargs):
98	            self = cls(**initkwargs)
99	            self.setup(request, *args, **kwargs)
100	            if not hasattr(self, "request"):
101	                raise AttributeError(
102	                    "%s instance has no 'request' attribute. Did you override "
103	                    "setup() and forget to call super()?" % cls.__name__
104	                )
105	            return self.dispatch(request, *args, **kwargs)
106	
107	        view.view_class = cls
108	        view.view_initkwargs = initkwargs

... (gap) ...

115	        view.__annotations__ = cls.dispatch.__annotations__
116	        # Copy possible attributes set by decorators, e.g. @csrf_exempt, from
117	        # the dispatch method.
118	        view.__dict__.update(cls.dispatch.__dict__)
119	
120	        # Mark the callback if the view class is async.
121	        if cls.view_is_async:
122	            markcoroutinefunction(view)
123	
124	        return view
125	
126	    def setup(self, request, *args, **kwargs):
127	        """Initialize attributes shared by all view methods."""
128	        if hasattr(self, "get") and not hasattr(self, "head"):
129	            self.head = self.get
130	        self.request = request
131	        self.args = args
132	        self.kwargs = kwargs
133	
134	    def dispatch(self, request, *args, **kwargs):
135	        # Try to dispatch to the right method; if a method doesn't exist,
136	        # defer to the error handler. Also defer to the error handler if the
137	        # request method isn't on the approved list.
138	        method = request.method.lower()
139	        if method in self.http_method_names:
140	            handler = getattr(self, method, self.http_method_not_allowed)
141	        else:
142	            handler = self.http_method_not_allowed
143	        return handler(request, *args, **kwargs)
144	
145	    def http_method_not_allowed(self, request, *args, **kwargs):
146	        response = HttpResponseNotAllowed(self._allowed_methods())
147	        log_response(
148	            "Method Not Allowed (%s): %s",
149	            request.method,
150	            request.path,
151	            response=response,
152	            request=request,
153	        )
154	
155	        if self.view_is_async:
156	
157	            async def func():
158	                return response
159	
160	            return func()
161	        else:
162	            return response
163	
164	    def options(self, request, *args, **kwargs):
165	        """Handle responding to requests for the OPTIONS HTTP verb."""
166	        response = HttpResponse()
167	        response.headers["Allow"] = ", ".join(self._allowed_methods())
168	        response.headers["Content-Length"] = "0"
169	
170	        if self.view_is_async:
171	
172	            async def func():
173	                return response
174	
175	            return func()
176	        else:
177	            return response
178	
179	    def _allowed_methods(self):
180	        return [m.upper() for m in self.http_method_names if hasattr(self, m)]
181	
182	
183	class TemplateResponseMixin:
```

**`django/core/management/commands/shell.py`** — calls(calls), globals(calls), add_arguments(method), get_auto_imports(calls), get_auto_imports(method), handle(method), isatty(calls), select(calls), read(calls), CommandError(instantiates), +9 more

```python
12	from django.utils.module_loading import import_string as import_dotted_path
13	
14	
15	class Command(BaseCommand):
16	    help = (
17	        "Runs a Python interactive interpreter. Tries to use IPython or "
18	        "bpython, if one of them is available. Any standard input is executed "

... (gap: add_arguments (django/core/management/commands/shell.py:26)) ...

58	    def ipython(self, options):
59	        from IPython import start_ipython
60	
61	        start_ipython(argv=[], user_ns=self.get_namespace(**options))
62	
63	    def bpython(self, options):
64	        import bpython
65	
66	        bpython.embed(self.get_namespace(**options))
67	
68	    def python(self, options):
69	        import code
70	
71	        # Set up a dictionary to serve as the environment for the shell.
72	        imported_objects = self.get_namespace(**options)
73	
74	        # We want to honor both $PYTHONSTARTUP and .pythonrc.py, so follow
75	        # system conventions and get $PYTHONSTARTUP first then .pythonrc.py.
76	        if not options["no_startup"]:
77	            for pythonrc in OrderedSet(
78	                [os.environ.get("PYTHONSTARTUP"), os.path.expanduser("~/.pythonrc.py")]
79	            ):
80	                if not pythonrc:
81	                    continue
82	                if not os.path.isfile(pythonrc):
83	                    continue
84	                with open(pythonrc) as handle:
85	                    pythonrc_code = handle.read()
86	                # Match the behavior of the cpython shell where an error in
87	                # PYTHONSTARTUP prints an exception and continues.
88	                try:
89	                    exec(compile(pythonrc_code, pythonrc, "exec"), imported_objects)
90	                except Exception:
91	                    traceback.print_exc()
92	
93	        # By default, this will set up readline to do tab completion and to
94	        # read and write history to the .python_history file, but this can be
95	        # overridden by $PYTHONSTARTUP or ~/.pythonrc.py.
96	        try:
97	            hook = sys.__interactivehook__
98	        except AttributeError:
99	            # Match the behavior of the cpython shell where a missing
100	            # sys.__interactivehook__ is ignored.
101	            pass
102	        else:
103	            try:
104	                hook()
105	            except Exception:
106	                # Match the behavior of the cpython shell where an error in
107	                # sys.__interactivehook__ prints a warning and the exception
108	                # and continues.
109	                print("Failed calling sys.__interactivehook__")
110	                traceback.print_exc()
111	
```

**`django/contrib/staticfiles/storage.py`** — calls(calls), clean_name(calls), hash_key(calls), instantiates(instantiates), _url(calls), converter(references), references(references), _post_process(calls), _css_ignored_re(variable), __init__(method), +28 more

```python
148	                    template = self.default_template
149	                    ignored_re = _css_ignored_re
150	                compiled = re.compile(pattern, re.IGNORECASE)
151	                self._patterns.setdefault(extension, []).append(
152	                    (compiled, template, ignored_re)
153	                )
154	
155	    def file_hash(self, name, content=None):
156	        """
157	        Return a hash of the file with the given name and optional content.
158	        """
159	        if content is None:
160	            return None
161	        hasher = md5(usedforsecurity=False)
162	        for chunk in content.chunks():
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

... (gap: _url (django/contrib/staticfiles/storage.py:200)) ...

214	                hashed_name = hashed_name_func(*args)
215	
216	        final_url = super().url(hashed_name)
217	
218	        # Special casing for a @font-face hack, like url(myfont.eot?#iefix")
219	        # http://www.fontspring.com/blog/the-new-bulletproof-font-face-syntax
220	        query_fragment = "?#" in name  # [sic!]
221	        if fragment or query_fragment:
222	            urlparts = list(urlsplit(final_url))
223	            if fragment and not urlparts[4]:
224	                urlparts[4] = fragment
225	            if query_fragment and not urlparts[3]:
226	                urlparts[2] += "?"
227	            final_url = urlunsplit(urlparts)
228	
229	        return unquote(final_url)
230	
231	    def url(self, name, force=False):
232	        """
233	        Return the non-hashed URL in DEBUG mode.
234	        """
235	        return self._url(self.stored_name, name, force)
236	
237	    def get_ignored_blocks(self, content, pattern):
238	        """
```

**`django/template/context_processors.py`** — request(function), _get_val(function), csp(function)

```python
1	"""
2	A set of request processors that return dictionaries to be merged into a
3	template context. Each function takes the request object as its only parameter
4	and returns a dictionary to add to the context.
5	
6	These are referenced from the 'context_processors' option of the configuration
7	of a DjangoTemplates backend and used by RequestContext.
8	"""
9	
10	import itertools
11	
12	from django.conf import settings
13	from django.middleware.csp import get_nonce
14	from django.middleware.csrf import get_token
15	from django.utils.csp import CONTEXT_KEY as CSP_CONTEXT_KEY
16	from django.utils.functional import SimpleLazyObject, lazy
17	
18	
19	def csrf(request):
20	    """
21	    Context processor that provides a CSRF token, or the string 'NOTPROVIDED'
22	    if it has not been provided by either a view decorator or the middleware
23	    """
24	
25	    def _get_val():
26	        token = get_token(request)
27	        if token is None:
28	            # In order to be able to provide debugging info in the
29	            # case of misconfiguration, we use a sentinel value
30	            # instead of returning an empty dict.
31	            return "NOTPROVIDED"
32	        else:
33	            return token
34	
35	    return {"csrf_token": SimpleLazyObject(_get_val)}
36	
37	
38	def debug(request):
39	    """
40	    Return context variables helpful for debugging.
41	    """
42	    context_extras = {}
43	    if settings.DEBUG and request.META.get("REMOTE_ADDR") in settings.INTERNAL_IPS:
44	        context_extras["debug"] = True
45	        from django.db import connections
46	
47	        # Return a lazy reference that computes connection.queries on access,
48	        # to ensure it contains queries triggered after this function runs.
49	        context_extras["sql_queries"] = lazy(
50	            lambda: list(
51	                itertools.chain.from_iterable(
52	                    connections[x].queries for x in connections
53	                )
54	            ),
55	            list,
56	        )
57	    return context_extras
58	
59	
60	def i18n(request):
61	    from django.utils import translation
62	
63	    return {
64	        "LANGUAGES": settings.LANGUAGES,
65	        "LANGUAGE_CODE": translation.get_language(),
66	        "LANGUAGE_BIDI": translation.get_language_bidi(),
67	    }
68	
69	
70	def tz(request):
71	    from django.utils import timezone
72	
73	    return {"TIME_ZONE": timezone.get_current_timezone_name()}
74	
75	
76	def static(request):
77	    """
78	    Add static-related context variables to the context.
79	    """
80	    return {"STATIC_URL": settings.STATIC_URL}
81	
82	
83	def media(request):
84	    """
85	    Add media-related context variables to the context.
86	    """
87	    return {"MEDIA_URL": settings.MEDIA_URL}
88	
89	
90	def request(request):
91	    return {"request": request}
92	
93	
94	def csp(request):
95	    """
96	    Add the CSP nonce to the context.
97	    """
98	    return {CSP_CONTEXT_KEY: get_nonce(request)}
```

**`django/forms/widgets.py`** — SelectDateWidget(class), calls(calls), Select(extends), __deepcopy__(method), __init__(method), _choice_has_empty_value(calls), _choice_has_empty_value(method), ChoiceWidget(class), ChoiceWidget(extends), get_context(method), +8 more

```python
728	        return False
729	
730	
731	class ChoiceWidget(Widget):
732	    allow_multiple_selected = False
733	    input_type = None
734	    template_name = None
735	    option_template_name = None
736	    add_id_index = True
737	    checked_attribute = {"checked": True}
738	    option_inherits_attrs = True
739	
740	    def __init__(self, attrs=None, choices=()):
741	        super().__init__(attrs)
742	        self.choices = choices
743	
744	    def __deepcopy__(self, memo):
745	        obj = copy.copy(self)
746	        obj.attrs = self.attrs.copy()
747	        obj.choices = copy.copy(self.choices)
748	        memo[id(self)] = obj
749	        return obj
750	
751	    def subwidgets(self, name, value, attrs=None):
752	        """
753	        Yield all "subwidgets" of this widget. Used to enable iterating
754	        options from a BoundField for choice widgets.
755	        """
756	        value = self.format_value(value)
757	        yield from self.options(name, value, attrs)
758	
759	    def options(self, name, value, attrs=None):
760	        """Yield a flat list of options for this widget."""
761	        for group in self.optgroups(name, value, attrs):
762	            yield from group[1]
763	
764	    def optgroups(self, name, value, attrs=None):
765	        """Return a list of optgroups for this widget."""

... (gap: create_option (django/forms/widgets.py:804), get_context (django/forms/widgets.py:827), id_for_label (django/forms/widgets.py:834), value_from_datadict (django/forms/widgets.py:843), format_value (django/forms/widgets.py:852), choices (django/forms/widgets.py:861)) ...

866	        self._choices = normalize_choices(value)
867	
868	
869	class Select(ChoiceWidget):
870	    input_type = "select"
871	    template_name = "django/forms/widgets/select.html"
872	    option_template_name = "django/forms/widgets/select_option.html"
873	    add_id_index = False
874	    checked_attribute = {"selected": True}
875	    option_inherits_attrs = False
876	
877	    def get_context(self, name, value, attrs):
878	        context = super().get_context(name, value, attrs)
879	        if self.allow_multiple_selected:
880	            context["widget"]["attrs"]["multiple"] = True
881	        return context
882	
883	    @staticmethod
884	    def _choice_has_empty_value(choice):
885	        """Return True if the choice's value is empty string or None."""
886	        value, _ = choice
887	        return value is None or value == ""
888	
889	    def use_required_attribute(self, initial):
890	        """
891	        Don't render 'required' if the first <option> has a value, as that's
892	        invalid HTML.
893	        """
894	        use_required_attribute = super().use_required_attribute(initial)
895	        # 'required' is always okay for <select multiple>.
896	        if self.allow_multiple_selected:
897	            return use_required_attribute
898	
899	        first_choice = next(iter(self.choices), None)
900	        return (
901	            use_required_attribute
902	            and first_choice is not None
903	            and self._choice_has_empty_value(first_choice)
904	        )
905	
906	
907	class NullBooleanSelect(Select):
908	    """
909	    A Select Widget intended to be used with NullBooleanField.
910	    """

... (gap: __init__ (django/forms/widgets.py:912), format_value (django/forms/widgets.py:920), value_from_datadict (django/forms/widgets.py:934)) ...

946	        }.get(value)
947	
948	
949	class SelectMultiple(Select):
950	    allow_multiple_selected = True
951	
952	    def value_from_datadict(self, data, files, name):
953	        try:
954	            getter = data.getlist
955	        except AttributeError:
956	            getter = data.get
957	        return getter(name)
958	
959	    def value_omitted_from_data(self, data, files, name):
960	        # An unselected <select multiple> doesn't appear in POST data, so it's
961	        # never known if the value is actually omitted.
962	        return False
963	
964	
965	class RadioSelect(ChoiceWidget):
```

**`django/tasks/base.py`** — Task(class), call(method), func(calls)

```python
128	            )
129	        return result
130	
131	    def call(self, *args, **kwargs):
132	        if iscoroutinefunction(self.func):
133	            return async_to_sync(self.func)(*args, **kwargs)
134	        return self.func(*args, **kwargs)
135	
136	    async def acall(self, *args, **kwargs):
137	        if iscoroutinefunction(self.func):
```

**`django/contrib/staticfiles/handlers.py`** — calls(calls), request(references), get_response_async(method), serve(calls), get_response(method), references(references), serve(method)

```python
45	        relative_url = url.removeprefix(self.base_url.path)
46	        return url2pathname(relative_url)
47	
48	    def serve(self, request):
49	        """Serve the request path."""
50	        return serve(request, self.file_path(request.path), insecure=True)
51	
52	    def get_response(self, request):
53	        try:
54	            return self.serve(request)
55	        except Http404 as e:
56	            return response_for_exception(request, e)
57	
58	    async def get_response_async(self, request):
59	        try:
60	            return await sync_to_async(self.serve, thread_sensitive=False)(request)
61	        except Http404 as e:
62	            return await sync_to_async(response_for_exception, thread_sensitive=True)(
63	                request, e
64	            )
65	
66	
67	class StaticFilesHandler(StaticFilesHandlerMixin, WSGIHandler):

... (gap: __init__ (django/contrib/staticfiles/handlers.py:73), __call__ (django/contrib/staticfiles/handlers.py:78), ASGIStaticFilesHandler (django/contrib/staticfiles/handlers.py:84)) ...

100	        # Hand off to the main app
101	        return await self.application(scope, receive, send)
102	
103	    async def get_response_async(self, request):
104	        response = await super().get_response_async(request)
105	        response._resource_closers.append(request.close)
106	        # FileResponse is not async compatible.
107	        if response.streaming and not response.is_async:
108	            _iterator = response.streaming_content
109	
110	            async def awrapper():
111	                for part in await sync_to_async(list)(_iterator):
112	                    yield part
113	
114	            response.streaming_content = awrapper()
115	        return response
116	
```

**`django/contrib/gis/db/backends/mysql/operations.py`** — instantiates(instantiates), calls(calls), gis_operators(method), disallowed_aggregates(method), unsupported_functions(method), geo_db_type(method), get_distance(method), get_geometry_converter(method), references(references), spatial_aggregate_name(method), +9 more

```python
9	from django.utils.functional import cached_property
10	
11	
12	class MySQLOperations(BaseSpatialOperations, DatabaseOperations):
13	    name = "mysql"
14	    geom_func_prefix = "ST_"
15	
16	    Adapter = WKTAdapter
17	
18	    @cached_property
19	    def mariadb(self):
20	        return self.connection.mysql_is_mariadb
21	
22	    @cached_property
23	    def mysql(self):
24	        return not self.connection.mysql_is_mariadb
25	
26	    @cached_property
27	    def select(self):
28	        return self.geom_func_prefix + "AsBinary(%s)"
29	
30	    @cached_property
31	    def from_text(self):
32	        return self.geom_func_prefix + "GeomFromText"
33	
34	    @cached_property
35	    def collect(self):
```

**Not shown above — explore these names for their source**

- tests/forms_tests/widget_tests/test_select.py: test_constructor_attrs:78, test_choices_constructor:132, test_choices_constructor_generator:145, SelectTest:10, test_select.py:1
- ... and 43 more files
````

## 判定

- 字符预算：通过
- 事实覆盖率：失败
- 重复结果缩减：通过
- 综合 Gate：FAIL

> 本文件保存的是固定测试时的原始结果。重新运行后应导出到另一个目录，以便进行版本间对比。
