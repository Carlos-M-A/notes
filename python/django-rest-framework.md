# Django REST Framework

Django REST Framework notes

Other good notes and cheatsheets I found:
* [testdriven.io Spela 1](https://testdriven.io/blog/drf-views-part-1/)
* [testdriven.io Spela 2](https://testdriven.io/blog/drf-views-part-2/)
* [testdriven.io Spela 3](https://testdriven.io/blog/drf-views-part-3/)

[github Nifled](https://github.com/Nifled/drf-cheat-sheet)


## BASIC

DRF is composed of the following two components:
* __Serializers__ are used to: 
  * Convert Django querysets and model object instances to (serialization) and from (deserialization) python dictionary (that can then be easily rendered into JSON, XML, and other formats.).
  * As Django Form class does, they validate data
* __Views__ (along with ViewSets) are used to:
  * As they are similar to traditional Django views, handle RESTful HTTP requests and responses.
  * Use the serializers to validate incoming payloads and contain the necessary logic to return the response.
  * Views are coupled with routers, which map the views back to the exposed URLs.

## SERIALIZERS

Serializers are similiar to Django Forms in some aspects. To use them:

<img src="./images/drf-serializer-process.png"  width="300" height="350">

```python
#Serialize:
# Object instance -> Data dict
s = Serializer(instance)
s.data

#Deserialize (create):
# dict -> Object
s = Serializer(data={..})
s.is_valid()
instance = s.save()

#Validation:
s = Serializer(data={..})
if s.is_valid(raise_exception=False):
  s.validated_data
else:
  s.errors # dict(field=[problems..])

#Update:
s = Serializer(instance, validated_data, partial=True)
new = s.save()
```

To create a serializer, we have to create a class that extends:
* Serializer 
* ListSerializer
* ModelSerializer (o HyperlinkedModelSerializer)

<img src="./images/drf-serializers.png"  width="300" height="300">

### Serializer

__Fields__:
* __Common__: CharField, ChoiceField, IntegerField, BooleanField, DateTimefield, UUIDField, JSONField, EmailField, URLField, FileField, ...
* __Relationships__: RelatedField, PrimaryKeyRelatedField SlugRelatedField, HyperlinkedRelatedField

__Field args__:
* read_only: Boolean. Default False
* write_only: Boolean. Default False
* required: Boolean. Default True
* allow_null: Boolean. Default False
* default: Any value depending of the field. If set, don't use the arg 'required'
* source: String. The name of the attribute that will be used to populate the field. May be a method that only takes a self argument, such as `URLField(source='get_absolute_url')`, or may use dotted notation to traverse attributes, such as `EmailField(source='user.email')`.
* validators: List of validator functions, such as `IntegerField(validators=[multiple_of_ten, less_than_20000])`.

__Methods__ :
* `create()`: Always needs to be overrided
* `update()`: Always needs to be overrided
* `save()`: Only override it for special behavior
* `validate_<field_name>()`: Such as `validate_title()`. They should return the validated value or raise a `serializers.ValidationError`.
* `validate()`: If you need to validate some fields together
* `is_valid()`: Don't override

```python
class SimpleSerializer(serializers.Serializer):
  name = serializers.CharField(max_length=128)
  field2 = serializers.IntegerField(required=false)
  ...

  def create(self, validated_data):
    ...
    return MyModel.objects.create(**validated_data)

  def update(self, instance, validated_data):
    # if self.partial
    instance.name = validated_data.get('name', instance.name)
    instance.field2 = validated_data.get('field2', instance.field2) 
    ...
    instance.save()
    return instance

  def save(self, **kwargs):
    # instance = super().save(**kwargs)
    nm = self.validated_data.get('name')
    ...
    instance = model_object.save()
    return instance

  # def validate_<field>(self, name):
  def validate_name(self, name):
    if not name:
      raise serializers.ValidationError('Name required')
    ...
    return name

  def validate(self, data):
    nm = data.get('name')
    if not nm:
      raise serializers.ValidationError({'name': ..})
    ...
    return validated_data
```

### ListSerializer

The `ListSerializer` class provides the behavior for serializing and validating multiple objects at once. You won't typically need to use `ListSerializer` directly, but should instead simply pass `many=True` when instantiating a serializer.

When a serializer is instantiated and `many=True` is passed, a `ListSerializer` instance will be created. The serializer class then becomes a child of the parent `ListSerializer`

The following argument can also be passed to a ListSerializer field or a serializer that is passed many=True:
* `allow_empty`: Dafault True
* `max_length`
* `min_length`


### ModelSerializer

ModelSerializer classes are simply a shortcut for creating serializer classes:
* An automatically determined set of fields.
* Simple default implementations for the create() and update() methods.

```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ('id', 'title', 'text', 'created')
        read_only_fields = ('crated',)
```

### HyperlinkedModelSerializer
```python
class ArticleSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Article
        fields = ('id', 'title', 'text', 'created', 'comments')
        read_only_fields = ('comments',)
```

Other way:
```python
class ArticleSerializer(serializers.ModelSerializer):
    comments = serializers.HyperlinkedRelatedField(many=True, view_name='comment-detail', read_only=True)
    
    class Meta:
        model = Article
        fields = ('id', 'title', 'text', 'created', 'comments')
```

### Nested serialization

__General__:

```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = '__all__'
        depth = 2
```

__Explicit__:
```python
class CommentSerializer(serializers.ModelSerializer):
    article = ArticleSerializer()
    class Meta:
        model = Comment
        fields = '__all__'
```

## VIEWS - Function based 

Using the @api_view decorator

Other decorators: @csrf_exempt, @renderer_classes, @parser_classes, @authentication_classes, @throttle_classes, @permission_classes

```python
@api_view(('GET','POST'))
def note_list(request):
    if request.method == 'GET':
        notes = Note.objects.all()
        serializer = NoteSerializer(notes, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        serializer = NoteSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def note_detail(request, note_id):
    try:
        note = Note.objects.get(pk=note_id)
    except Note.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = NoteSerializer(note)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = NoteSerializer(note, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'PATCH':
        serializer = NoteSerializer(note, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        note.delete()
        return Response(status=status.HTTP_204_NO_CONTENT) 
```


## VIEWS - Class based

<img src="./images/drf-views.png"  width="800" height="700">

There are 4 ways of doing class based views
* To extend APIView
* To extend GenericAPIView and mixins
* To extend concrete views (generics) (like ListCreateAPIView, or RetrieveUpdateAPIView)
* To extend set views (like ModelViewSet or ReadOnlyModelViewSet)

 But in real life, you'll most likely use one of the following:: APIView, concrete views, and (ReadOnly)ModelViewSet 

Policy attributes:
* renderer_classes: JSONRenderer, BrowsableAPIRenderer, ...
* parser_classes: JSONParser, FileUploadParser, ...
* authentication_classes: TokenAuthentication, SessionAuthentication, ...
* permission_classes: IsAuthenticated, DjangoModelPermissions, ...
* throttle_classes: AnonRateThrottle, UserRateThrottle, ...
* content_negotiation_class: only custom content negotiation classes

Policy methods:
* check_permissions(self, request)
* check_object_permissions(self, request): In Generic Views or ViewSets
* check_throttles(self, request)
* perform_content_negotiation(self, request, force=False)

### APIView

```python
class NoteListApiView(APIView):
    def get(self, request, format=None):
        notes = Note.objects.all()
        serializer = NoteSerializer(notes, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = NoteSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class NoteDetailApiView(APIView):
    def get_object(self, note_id):
        try:
            return Note.objects.get(pk=note_id)
        except Note.DoesNotExist:
            raise Http404

    def get(self, request, note_id, format=None):
        note = self.get_object(note_id)
        serializer = NoteSerializer(note)
        return Response(serializer.data)

    def put(self, request, note_id, format=None):
        note = self.get_object(note_id)
        serializer = NoteSerializer(note, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def patch(self, request, note_id, format=None):
        note = self.get_object(note_id)
        serializer = NoteSerializer(note, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, note_id, format=None):
        note = self.get_object(note_id)
        note.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```


### Mixins

Attributtes:
* queryset
* serializer_class

Methods:
* get_queryset()
* get_object()
* get_serializer()

```python
class NoteList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Note.objects.all()
    serializer_class = NoteSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

class NoteDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Note.objects.all()
    serializer_class = NoteSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```


### Concrete Views

* CreateAPIView: POST (single)
* ListAPIView: GET (multiple)
* RetrieveAPIView: GET (single)
* DestroyAPIView: DELETE (single)
* UpdateAPIView: PUT/PATCH (single)
* ListCreateAPIView: POST (multiple)
* RetrieveUpdateAPIView: GET/PUT/PATCH (single)
* RetrieveDestroyAPIView: GET/DELETE (single)
* RetrieveUpdateDestroyAPIView: GET/PUT/PATCH/DELETE (single)

```python
class NoteList(generics.ListCreateAPIView):
    queryset = Note.objects.all()
    serializer_class = NoteSerializer


class NoteDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Note.objects.all()
    serializer_class = NoteSerializer
```


## Viewsets

* ViewSet
* GenericViewSet
* ReadOnlyModelViewSet
* ModelViewSet

Actions:
* list: GET (multiple)
* create: POST
* retrieve: GET (single) (pk needed)
* update: PUT (pk needed)
* partial_update: PATCH (pk needed)
* destroy: DELETE (pk needed)

You can also create custom actions with the `@action` decorator.


```python
class NoteViewSet(viewsets.ModelViewSet):
    """
    This viewset automatically provides `list`, `create`, `retrieve`,
    `update` and `destroy` actions.
    """
    queryset = Note.objects.all()
    serializer_class = NoteSerializer
```

Then in urls.py:

```python
note_list = NoteViewSet.as_view({
    'get': 'list',
    'post': 'create'
})
note_detail = NoteViewSet.as_view({
    'get': 'retrieve',
    'put': 'update',
    'patch': 'partial_update',
    'delete': 'destroy'
})
path('notes/', note_list, name='note-list'),
path('notes/<int:pk>/', note_detail, name='note-detail'),
```

### Routers
 ```python
router = routers.DefaultRouter()
router.register('notes', NoteViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```


## Requests and Responses

DRF Request extends HttpRequest
```python
request.POST  # Only handles form data.  Only works for 'POST' method.
request.data  # Handles arbitrary data.  Works for 'POST', 'PUT' and 'PATCH' methods.
```

DRF Response extends HttpResponse
Response object is a type of TemplateResponse that takes unrendered content and uses content negotiation to determine the correct content type to return to the client.
```python
if serializer.is_valid():
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

## TESTING

