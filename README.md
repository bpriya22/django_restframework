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
   class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']

3. One nice property that serializers have is that you can inspect all the fields in a serializer instance, by printing its representation.

   ex: in python shell:
        type the following:
        from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
# SnippetSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    title = CharField(allow_blank=True, max_length=100, required=False)
#    code = CharField(style={'base_template': 'textarea.html'})
#    linenos = BooleanField(required=False)
#    language = ChoiceField(choices=[('Clipper', 'FoxPro'), ('Cucumber', 'Gherkin'), ('RobotFramework', 'RobotFramework'), ('abap', 'ABAP'), ('ada', 'Ada')...
#    style = ChoiceField(choices=[('autumn', 'autumn'), ('borland', 'borland'), ('bw', 'bw'), ('colorful', 'colorful')...

3. It's important to remember that ModelSerializer classes don't do anything particularly magical, they are simply a shortcut for creating serializer classes:

     a. An automatically determined set of fields.
     b. Simple default implementations for the create() and update() methods.

Writing regular Django views using our Serializer
=================================================

1. writing api views using our new serializer.

@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)

 2. Note that because we want to be able to POST to this view from clients that won't have a CSRF token we need to mark the view as csrf_exempt.

 Requests and Responses
=

1. request.POST  # Only handles form data.  Only works for 'POST' method.
request.data  # Handles arbitrary data.  Works for 'POST', 'PUT' and 'PATCH' methods.

2. return Response(data)  # Renders to content type as requested by the client.

Wrapping API views
=

1. The @api_view decorator for working with function based views.
2. The APIView class for working with class-based views.
3. Notice that we're no longer explicitly tying our requests or responses to a given content type. request.data can handle incoming json requests, but it can also handle other formats. Similarly we're returning response objects with data, but allowing REST framework to render the response into the correct content type for us.

Adding optional format suffixes to our URLs
=

1. Using format suffixes gives us URLs that explicitly refer to a given format, and means our API will be able to handle URLs such as http://example.com/api/items/4.json.
2. example:  def snippet_list(request, format=None):
3. Now update the snippets/urls.py file slightly, to append a set of format_suffix_patterns in addition to the existing URLs.

from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
We don't necessarily need to add these extra url patterns in, but it gives us a simple, clean way of referring to a specific format.
4. We can control the format of the response that we get back, either by using the Accept header:

http http://127.0.0.1:8000/snippets/ Accept:application/json  # Request JSON
http http://127.0.0.1:8000/snippets/ Accept:text/html         # Request HTML
