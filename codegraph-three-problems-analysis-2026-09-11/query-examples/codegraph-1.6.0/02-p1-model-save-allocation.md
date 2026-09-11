# Model.save：大型核心文件中的内容分配

- 用例 ID：`p1-model-save-allocation`
- 对应问题：P1
- CodeGraph：1.6.0
- Django 项目：`D:\工作空间\harbor\exp\django`
- 记录时间：2026-09-10T12:25:23.857Z
- 用例配置哈希：`早期记录未提供`

## 测试目的

django/db/models/base.py is large; a useful response must prioritize the save spine instead of spending the budget on nearby methods.

## 给 Agent 的任务

```text
Trace Django Model.save from its public entry point to the update-versus-insert decision. Identify the core methods in order and explain where the branch occurs. Keep the answer focused on the persistence spine.
```

## 直接 MCP 查询

### 第 1 次调用

```text
Model.save save_base _save_table _do_update _do_insert
```

复现命令：

```powershell
codegraph explore "Model.save save_base _save_table _do_update _do_insert" --path "D:\工作空间\harbor\exp\django"
```

| 指标 | 结果 |
|---|---:|
| 响应字符数 | 24862 |
| 估算 token | 8287 |
| 查询耗时 | 417.1 ms |
| 事实覆盖率 | 1 |
| 概念命中 | 6/6 |
| 文件命中 | 1/1 |
| MCP 错误 | 否 |

#### 完整原始返回

````text
**Flow (call path among the symbols you queried)**

1. save (django/db/models/base.py:847)
   ↓ calls (when !(force_insert and (force_update or update_fields)))
2. save_base (django/db/models/base.py:956)
   ↓ calls
3. _save_table (django/db/models/base.py:1075)
   ↓ calls (when !(not pk_set and (force_update or update_fields)) && pk_set and not force_insert)
4. _do_update (django/db/models/base.py:1214)

> Full source for these symbols is below — the call flow among them, followed by their bodies.
**Exploration: Model.save save_base _save_table _do_update _do_insert**

Found 65 symbols across 8 files.

**Blast radius — what depends on these (update/verify before editing)**

- `ModelBase` (django/db/models/base.py:102) — 8 callers in `django/contrib/admin/sites.py`, `django/contrib/contenttypes/fields.py`, `django/db/models/base.py`; tested via callers: `tests/admin_default_site/tests.py`, `tests/admin_registration/tests.py` +54
- `_save_table` (django/db/models/base.py:1075) — 2 callers in `django/db/models/base.py`; tested via callers: `tests/model_inheritance_regress/tests.py`, `tests/serializers/tests.py` +2
- `save_base` (django/db/models/base.py:956) — 15 callers in `django/core/serializers/base.py`, `django/db/models/base.py`; tests: `tests/model_inheritance_regress/tests.py`, `tests/serializers/tests.py`, `tests/serializers/test_data.py`, `tests/signals/tests.py`
- `insert` (django/contrib/gis/geos/mutable_list.py:196) — 20 callers in `django/contrib/admin/filters.py`, `django/contrib/contenttypes/management/__init__.py`, `django/core/handlers/base.py`, `django/core/management/base.py` +5 more; tests: `django/test/utils.py`, `tests/file_uploads/views.py`, `tests/sphinx_tests/tests.py`, `tests/test_runner/tests.py` +1
- `save` (django/contrib/auth/forms.py:444) — 13 callers; tests: `tests/auth_tests/test_forms.py`

**Relationships**

**references:**
- register → ModelBase
- unregister → ModelBase
- _check_generic_foreign_key_existence → ModelBase
- __new__ → ModelBase
- _validate_force_insert → ModelBase
- _check_m2m_through_same_relationship → ModelBase
- __new__ → OneToOneField
- _save_table → DatabaseDefault
- save → UserModel
- save → _save
- ... and 1 more

**calls:**
- __new__ → _has_contribute_to_class
- __new__ → get_containing_app_config
- __new__ → add_to_class
- __new__ → subclass_exception
- __new__ → chain
- __new__ → setup_proxy
- __new__ → resolve_relation
- __new__ → make_model_tuple
- __new__ → copy
- __new__ → update
- ... and 107 more

**instantiates:**
- __new__ → Options
- __new__ → FieldError
- __new__ → OneToOneField
- _save_table → Coalesce
- _save_table → ExpressionWrapper
- _save_table → Max
- _save_table → Value
- _save_table → IntegerField
- test_user_email_unicode_collision → PasswordResetForm
- test_user_email_domain_unicode_collision → PasswordResetForm
- ... and 8 more

**extends:**
- SessionManager → BaseSessionManager
- Session → AbstractBaseSession

**decorates:**
- model → cached_property

**Source Code**

> The code below is the **verbatim, current on-disk source** of these files — re-read from disk on this call and line-numbered, byte-for-byte identical to what the Read tool returns. It is NOT a summary, outline, or stale cache. Treat each block as a Read you have already performed: do not Read a file shown here.

**`django/db/models/base.py`** — calls(calls), subclass_exception(calls), FieldError(instantiates), _is_pk_set(calls), _has_contribute_to_class(calls), references(references), _get_pk_val(calls), using(calls), _assign_returned_values(calls), Value(instantiates), +58 more

```python
99	    return not inspect.isclass(value) and hasattr(value, "contribute_to_class")
100	
101	
102	class ModelBase(type):
103	    """Metaclass for all models."""
104	
105	    def __new__(cls, name, bases, attrs, **kwargs):
106	        super_new = super().__new__
107	
108	        # Also ensure initialization is only performed for subclasses of Model
109	        # (excluding Model class itself).
110	        parents = [b for b in bases if isinstance(b, ModelBase)]
111	        if not parents:
112	            return super_new(cls, name, bases, attrs)
113	
114	        # Create the class.
115	        module = attrs.pop("__module__")
116	        new_attrs = {"__module__": module}
117	        classcell = attrs.pop("__classcell__", None)
118	        if classcell is not None:
119	            new_attrs["__classcell__"] = classcell
120	        attr_meta = attrs.pop("Meta", None)
121	        # Pass all attrs without a (Django-specific) contribute_to_class()
122	        # method to type.__new__() so that they're properly initialized
123	        # (i.e. __set_name__()).
124	        contributable_attrs = {}
125	        for obj_name, obj in attrs.items():
126	            if _has_contribute_to_class(obj):
127	                contributable_attrs[obj_name] = obj
128	            else:
129	                new_attrs[obj_name] = obj
130	        new_class = super_new(cls, name, bases, new_attrs, **kwargs)
131	
132	        abstract = getattr(attr_meta, "abstract", False)
133	        meta = attr_meta or getattr(new_class, "Meta", None)
134	        base_meta = getattr(new_class, "_meta", None)
135	
136	        app_label = None
137	
138	        # Look for an application configuration to attach the model to.
139	        app_config = apps.get_containing_app_config(module)
140	
141	        if getattr(meta, "app_label", None) is None:
142	            if app_config is None:
143	                if not abstract:
144	                    raise RuntimeError(
145	                        "Model class %s.%s doesn't declare an explicit "
146	                        "app_label and isn't in an application in "
147	                        "INSTALLED_APPS." % (module, name)
148	                    )
149	
150	            else:
151	                app_label = app_config.label
152	
153	        new_class.add_to_class("_meta", Options(meta, app_label))
154	        if not abstract:
155	            new_class.add_to_class(
156	                "DoesNotExist",
157	                subclass_exception(
158	                    "DoesNotExist",
159	                    tuple(
160	                        x.DoesNotExist
161	                        for x in parents
162	                        if hasattr(x, "_meta") and not x._meta.abstract
163	                    )
164	                    or (ObjectDoesNotExist,),
165	                    module,
166	                    attached_to=new_class,
167	                ),
168	            )
169	            new_class.add_to_class(
170	                "MultipleObjectsReturned",
171	                subclass_exception(
172	                    "MultipleObjectsReturned",
173	                    tuple(
174	                        x.MultipleObjectsReturned
175	                        for x in parents
176	                        if hasattr(x, "_meta") and not x._meta.abstract
177	                    )
178	                    or (MultipleObjectsReturned,),
179	                    module,
180	                    attached_to=new_class,
181	                ),
182	            )
183	            new_class.add_to_class(
184	                "NotUpdated",
185	                subclass_exception(
186	                    "NotUpdated",
187	                    tuple(
188	                        x.NotUpdated
189	                        for x in parents
190	                        if hasattr(x, "_meta") and not x._meta.abstract
191	                    )
192	                    # Subclass DatabaseError as well for backward compatibility
193	                    # reasons as __subclasshook__ is not taken into account on
194	                    # exception handling.
195	                    or (ObjectNotUpdated, DatabaseError),
196	                    module,
197	                    attached_to=new_class,
198	                ),
199	            )
200	            if base_meta and not base_meta.abstract:
201	                # Non-abstract child classes inherit some attributes from their
202	                # non-abstract parent (unless an ABC comes before it in the
203	                # method resolution order).
204	                if not hasattr(meta, "ordering"):
205	                    new_class._meta.ordering = base_meta.ordering
206	                if not hasattr(meta, "get_latest_by"):
207	                    new_class._meta.get_latest_by = base_meta.get_latest_by
208	
209	        is_proxy = new_class._meta.proxy
210	
211	        # If the model is a proxy, ensure that the base class
212	        # hasn't been swapped out.
213	        if is_proxy and base_meta and base_meta.swapped:
214	            raise TypeError(
215	                "%s cannot proxy the swapped model '%s'." % (name, base_meta.swapped)
216	            )
217	
218	        # Add remaining attributes (those with a contribute_to_class() method)
219	        # to the class.
220	        for obj_name, obj in contributable_attrs.items():
221	            new_class.add_to_class(obj_name, obj)
222	
223	        # All the fields of any type declared on this model
224	        new_fields = chain(
225	            new_class._meta.local_fields,
226	            new_class._meta.local_many_to_many,
227	            new_class._meta.private_fields,
228	        )
229	        field_names = {f.name for f in new_fields}
230	
231	        # Basic setup for proxy models.
232	        if is_proxy:
233	            base = None
234	            for parent in [kls for kls in parents if hasattr(kls, "_meta")]:
235	                if parent._meta.abstract:
236	                    if parent._meta.fields:
237	                        raise TypeError(
238	                            "Abstract base class containing model fields not "
239	                            "permitted for proxy model '%s'." % name
240	                        )
241	                    else:
242	                        continue
243	                if base is None:
244	                    base = parent
245	                elif parent._meta.concrete_model is not base._meta.concrete_model:
246	                    raise TypeError(
247	                        "Proxy model '%s' has more than one non-abstract model base "
248	                        "class." % name
249	                    )
250	            if base is None:
251	                raise TypeError(
252	                    "Proxy model '%s' has no non-abstract model base class." % name
253	                )
254	            new_class._meta.setup_proxy(base)
255	            new_class._meta.concrete_model = base._meta.concrete_model
256	        else:
257	            new_class._meta.concrete_model = new_class
258	
259	        # Collect the parent links for multi-table inheritance.
260	        parent_links = {}
261	        for base in reversed([new_class, *parents]):
262	            # Conceptually equivalent to `if base is Model`.
263	            if not hasattr(base, "_meta"):
264	                continue
265	            # Skip concrete parent classes.
266	            if base != new_class and not base._meta.abstract:
267	                continue
268	            # Locate OneToOneField instances.
269	            for field in base._meta.local_fields:
270	                if isinstance(field, OneToOneField) and field.remote_field.parent_link:
271	                    related = resolve_relation(new_class, field.remote_field.model)
272	                    parent_links[make_model_tuple(related)] = field
273	
274	        # Track fields inherited from base models.

... (gap: add_to_class (django/db/models/base.py:399), _prepare (django/db/models/base.py:405), _base_manager (django/db/models/base.py:461), _default_manager (django/db/models/base.py:465), ModelStateFieldsCacheDescriptor (django/db/models/base.py:469), __get__ (django/db/models/base.py:470), +21 more) ...

844	            return getattr(self, field_name)
845	        return getattr(self, field.attname)
846	
847	    def save(
848	        self,
849	        *,
850	        force_insert=False,
851	        force_update=False,
852	        using=None,
853	        update_fields=None,
854	    ):
855	        """
856	        Save the current instance. Override this in a subclass if you want to
857	        control the saving process.
858	
859	        The 'force_insert' and 'force_update' parameters can be used to insist
860	        that the "save" must be an SQL insert or update (or equivalent for
861	        non-SQL backends), respectively. Normally, they should not be set.
862	        """
863	
864	        self._prepare_related_fields_for_save(operation_name="save")
865	
866	        using = using or router.db_for_write(self.__class__, instance=self)
867	        if force_insert and (force_update or update_fields):
868	            raise ValueError("Cannot force both insert and updating in model saving.")
869	
870	        deferred_non_generated_fields = {
871	            f.attname
872	            for f in self._meta.concrete_fields
873	            if f.attname not in self.__dict__ and f.generated is False
874	        }
875	        if update_fields is not None:
876	            # If update_fields is empty, skip the save. We do also check for
877	            # no-op saves later on for inheritance cases. This bailout is
878	            # still needed for skipping signal sending.
879	            if not update_fields:
880	                return
881	
882	            update_fields = frozenset(update_fields)
883	            field_names = self._meta._non_pk_concrete_field_names
884	            not_updatable_fields = update_fields.difference(field_names)
885	
886	            if not_updatable_fields:
887	                raise ValueError(
888	                    "The following fields do not exist in this model, are m2m "
889	                    "fields, primary keys, or are non-concrete fields: %s"
890	                    % ", ".join(not_updatable_fields)
891	                )
892	
893	        # If saving to the same database, and this model is deferred, then
894	        # automatically do an "update_fields" save on the loaded fields.
895	        elif (
896	            not force_insert
897	            and deferred_non_generated_fields
898	            and using == self._state.db
899	            and self._is_pk_set()
900	        ):
901	            field_names = set()
902	            pk_fields = self._meta.pk_fields
903	            for field in self._meta.concrete_fields:
904	                if field not in pk_fields and not hasattr(field, "through"):
905	                    field_names.add(field.attname)
906	            loaded_fields = field_names.difference(deferred_non_generated_fields)
907	            if loaded_fields:
908	                update_fields = frozenset(loaded_fields)
909	
910	        self.save_base(
911	            using=using,
912	            force_insert=force_insert,
913	            force_update=force_update,
914	            update_fields=update_fields,
915	        )
916	
917	    save.alters_data = True
918	

... (gap: asave (django/db/models/base.py:919)) ...

937	    def _validate_force_insert(cls, force_insert):
938	        if force_insert is False:
939	            return ()
940	        if force_insert is True:
941	            return (cls,)

... (gap: save_base (django/db/models/base.py:956)) ...

974	        assert not (force_insert and (force_update or update_fields))
975	        assert update_fields is None or update_fields
976	        cls = origin = self.__class__
977	        # Skip proxies, but keep the origin as the proxy model.
978	        if cls._meta.proxy:
979	            cls = cls._meta.concrete_model
980	        meta = cls._meta
981	        if not meta.auto_created:
982	            pre_save.send(
983	                sender=origin,
984	                instance=self,
985	                raw=raw,
986	                using=using,
987	                update_fields=update_fields,
988	            )
989	        # A transaction isn't needed if one query is issued.
990	        if meta.parents:
991	            context_manager = transaction.atomic(using=using, savepoint=False)
992	        else:
993	            context_manager = transaction.mark_for_rollback_on_error(using=using)
994	        with context_manager:
995	            parent_inserted = False
996	            if not raw:
997	                # Validate force insert only when parents are inserted.
998	                force_insert = self._validate_force_insert(force_insert)
999	                parent_inserted = self._save_parents(
1000	                    cls, using, update_fields, force_insert
1001	                )
1002	            updated = self._save_table(
1003	                raw,
1004	                cls,
1005	                force_insert or parent_inserted,
1006	                force_update,
1007	                using,
1008	                update_fields,
1009	            )
1010	        # Store the database on which the object was saved
1011	        self._state.db = using
1012	        # Once saved, this is no longer a to-be-added instance.
1013	        self._state.adding = False
1014	
1015	        # Signal that the save is complete
1016	        if not meta.auto_created:
1017	            post_save.send(
1018	                sender=origin,
1019	                instance=self,
1020	                created=(not updated),
1021	                update_fields=update_fields,
1022	                raw=raw,
1023	                using=using,
1024	            )
1025	
1026	    save_base.alters_data = True
1027	
1028	    def _save_parents(
1029	        self, cls, using, update_fields, force_insert, updated_parents=None
1030	    ):
```

**`django/db/models/expressions.py`** — copy, ExpressionWrapper, Value, DatabaseDefault · skeleton (signatures only — codegraph_explore a name for its full body; do NOT Read)

```python
443	def copy(self):
1148	class Value(Expression):
1311	class DatabaseDefault(Expression):
1548	class ExpressionWrapper(SQLiteNumericMixin, Expression):
```

**`django/contrib/sites/requests.py`** — save(method), RequestSite(class), __init__(method), __str__(method), delete(method)

```python
1	class RequestSite:
2	    """
3	    A class that shares the primary interface of Site (i.e., it has ``domain``
4	    and ``name`` attributes) but gets its data from an HttpRequest object
5	    rather than from a database.
6	
7	    The save() and delete() methods raise NotImplementedError.
8	    """
9	
10	    def __init__(self, request):
11	        self.domain = self.name = request.get_host()
12	
13	    def __str__(self):
14	        return self.domain
15	
16	    def save(self, force_insert=False, force_update=False):
17	        raise NotImplementedError("RequestSite cannot be saved.")
18	
19	    def delete(self):
20	        raise NotImplementedError("RequestSite cannot be deleted.")
```

**`django/contrib/gis/utils/layermapping.py`** — calls(calls), instantiates(instantiates), _save(calls), LayerMapError(class), LayerMapError(instantiates), references(references), check_fid_range(calls), check_fid_range(method), feature_kwargs(calls), feature_kwargs(method), +4 more

```python
557	            and model_field.__class__.__name__ == "Multi%s" % geom_type.django
558	        )
559	
560	    def save(
561	        self,
562	        verbose=False,
563	        fid_range=False,
564	        step=False,
565	        progress=False,
566	        silent=False,
567	        stream=sys.stdout,
568	        strict=False,
569	    ):
570	        """
571	        Save the contents from the OGR DataSource Layer into the database
572	        according to the mapping dictionary given at initialization.
573	
574	        Keyword Parameters:
575	         verbose:
576	           If set, information will be printed subsequent to each model save
577	           executed on the database.
578	
579	         fid_range:
580	           May be set with a slice or tuple of (begin, end) feature ID's to map
581	           from the data source. In other words, this keyword enables the user
582	           to selectively import a subset range of features in the geographic
583	           data source.
584	
585	         step:
586	           If set with an integer, transactions will occur at every step
587	           interval. For example, if step=1000, a commit would occur after
588	           the 1,000th feature, the 2,000th feature etc.
589	
590	         progress:
591	           When this keyword is set, status information will be printed giving
592	           the number of features processed and successfully saved. By default,
593	           progress information will be printed every 1000 features processed,
```

**`django/contrib/sessions/base_session.py`** — get_decoded(method), get_session_store_class(calls), __str__(method), AbstractBaseSession(class), BaseSessionManager(class), BaseSessionManager(instantiates), encode(calls), encode(method), get_session_store_class(method), Meta(class), +2 more

```python
8	
9	
10	class BaseSessionManager(models.Manager):
11	    def encode(self, session_dict):
12	        """
13	        Return the given session dictionary serialized and encoded as a string.
14	        """
15	        session_store_class = self.model.get_session_store_class()
16	        return session_store_class().encode(session_dict)
17	
18	    def save(self, session_key, session_dict, expire_date):
19	        s = self.model(session_key, self.encode(session_dict), expire_date)
20	        if session_dict:
21	            s.save()
22	        else:
23	            s.delete()  # Clear sessions with no data.
24	        return s
25	
26	
27	class AbstractBaseSession(models.Model):
28	    session_key = models.CharField(_("session key"), max_length=40, primary_key=True)
29	    session_data = models.TextField(_("session data"))
30	    expire_date = models.DateTimeField(_("expire date"), db_index=True)
31	
32	    objects = BaseSessionManager()
33	
34	    class Meta:
35	        abstract = True
36	        verbose_name = _("session")
37	        verbose_name_plural = _("sessions")
38	
39	    def __str__(self):
```

**`django/contrib/auth/forms.py`** — calls(calls), get_email_field_name(calls), UserModel(variable), PasswordResetForm(class), send_mail(calls), send_mail(method), instantiates(instantiates), get_users(calls), get_users(method), force_bytes(calls), +5 more

```python
441	            and _unicode_ci_compare(email, getattr(u, email_field_name))
442	        )
443	
444	    def save(
445	        self,
446	        domain_override=None,
447	        subject_template_name="registration/password_reset_subject.txt",
448	        email_template_name="registration/password_reset_email.html",
449	        use_https=False,
450	        token_generator=default_token_generator,
451	        from_email=None,
452	        request=None,
453	        html_email_template_name=None,
454	        extra_email_context=None,
455	    ):
456	        """
457	        Generate a one-use only link for resetting password and send it to the
458	        user.
459	        """
460	        email = self.cleaned_data["email"]
461	        if not domain_override:
462	            current_site = get_current_site(request)
463	            site_name = current_site.name
464	            domain = current_site.domain
465	        else:
466	            site_name = domain = domain_override
467	        email_field_name = UserModel.get_email_field_name()
468	        for user in self.get_users(email):
```

**`django/contrib/gis/geos/mutable_list.py`** — __init__(method), __getitem__(method), calls(calls), __delitem__(method), references(references), __setitem__(method), insert(method)

```python
193	        "Standard list extend method"
194	        self[len(self) :] = vals
195	
196	    def insert(self, index, val):
197	        "Standard list insert method"
198	        if not isinstance(index, int):
199	            raise TypeError("%s is not a legal index" % index)
200	        self[index:index] = [val]
201	
202	    def pop(self, index=-1):
203	        "Standard list pop method"
```

**`django/db/models/fields/__init__.py`** — get_pk_value_on_save, has_default, has_db_default, get_filter_kwargs_for_object, pre_save, IntegerField · skeleton (signatures only — codegraph_explore a name for its full body; do NOT Read)

```python
788	def get_pk_value_on_save(self, instance):
1009	def get_filter_kwargs_for_object(self, obj):
1027	def pre_save(self, model_instance, add):
1054	def has_default(self):
1058	def has_db_default(self):
2142	class IntegerField(Field):
```

````

## 判定

- 字符预算：通过
- 事实覆盖率：通过
- 重复结果缩减：通过
- 综合 Gate：PASS

> 本文件保存的是固定测试时的原始结果。重新运行后应导出到另一个目录，以便进行版本间对比。
