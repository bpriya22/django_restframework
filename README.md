SERIALIZATION-https://www.django-rest-framework.org/tutorial/1-serialization/


Note
1. A serializer class is very similar to a Django Form class, and includes similar validation flags on the various fields, such as required, max_length and default.
2. The field flags can also control how the serializer should be displayed in certain circumstances, such as when rendering to HTML. The {'base_template': 'textarea.html'} flag 
 above is equivalent to using widget=widgets.Textarea on a Django Form class


3. 
   from snippets.models import Snippet
>>> from snippets.serializers import SnippetSerializer
>>> from rest_framework.renderers import JSONRenderer
>>> from rest_framework.parsers import JSONParser
>>> 
>>> snippet = Snippet(code='foo = "bar"\n')
>>> snippet.save()
>>> 
>>> snippet = Snippet(code='print("hello, world")\n')
>>> snippet.save()
>>> 
>>> serializer=SnippetSerializer(snippet)
>>> serializer.data
{'id': 4, 'title': '', 'code': 'print("hello, world")\n', 'linenos': False, 'language': 'python', 'style': 'friendly'}


Process of serialization:
========================
1. At this point we've translated the model instance into Python native datatypes. To finalize the serialization process we render the data into json.

2. ex 1:
         serializer = SnippetSerializer(snippet)
serializer.data
# {'id': 2, 'title': '', 'code': 'print("hello, world")\n', 'linenos': False, 'language': 'python', 'style': 'friendly'}

3. ex 2:- Process of serializing starts here...
     json data--

content = JSONRenderer().render(serializer.data)
content
# b'{"id": 2, "title": "", "code": "print(\\"hello, world\\")\\n", "linenos": false, "language": "python", "style": "friendly"}'

4. Process of deserializing:
   =========================

   Deserialization is similar. First we parse a stream into Python native datatypes...
   ...then we restore those native datatypes into a fully populated object instance.

5. We can also serialize querysets instead of model instances. To do so we simply add a many=True flag to the serializer arguments.
   ===
6. to serialize querysets..
   =======================
   We can also serialize querysets instead of model instances. To do so we simply add a many=True flag to the serializer arguments.
   serializer = SnippetSerializer(Snippet.objects.all(), many=True)                     =========
    serializer.data
    # [OrderedDict([('id', 1), ('title', ''), ('code', 'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2),       ('title', ''), ('code', 'print("hello, world")\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', ''), ('code', 'print("hello, world")'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]


Using ModelSerializer
===
1. In the same way that Django provides both Form classes and ModelForm classes, REST framework includes both Serializer classes, and ModelSerializer classes.
                                             ============     =================                               ==================      =======================

2. 
