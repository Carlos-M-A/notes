# Django

Django basic notes


## COMMANDS

Adding 'python manage.py' ahead:

* startproject <project_name>
* startapp <app_name>
* makemigrations
* migrate
* runserver <addr:port>
* test
* collectstatic
* check
* createsuperuser
* changepassword <username>

## AUTHENTICATION

### User class & methods
* Fields for data: `username, first_name, last_name, email, password, last_login, date_joined`
* Fields for security: `groups, user_permissions, is_staff, is_active`
* Attributes (always same value): `is_authenticated = True, is_anonymous = False`
* Methods (remember to use instead of fields directly):
  * `get_username()`
  * `set_password(raw_password)`
  * `check_password(raw_password)`

>Remember to use always ` User =  django.contrib.auth.get_user_model()` to get the User class.


### Django auth

#### Methods
* `authenticate(request, username, password) -> User`
* `login(request, username)`
* `logout(request)`
* `logout_then_login(request)`
* `redirect_to_login(next)`


#### Default AUTH: views - forms - urls - template names

| **url names**		  	| **views**                 	| **forms**          	| **templates**                	|
|-------------------------	|---------------------------	|--------------------	|------------------------------	|
| login                   	| LoginView                 	| AuthenticationForm 	| login.html                   	|
| logout                  	| LogoutView                	|                    	| logged_out.html              	|
| password_change         	| PasswordChangeView        	| PasswordChangeForm 	| password_change_form.html    	|
| password_change_done    	| PasswordChangeDoneView    	|                    	| password_change_done.html    	|
| password_reset          	| PasswordResetView         	| PasswordResetForm  	| password_reset_form.html     	|
| password_reset_done     	| PasswordResetDoneView     	|                    	| password_reset_done.html     	|
| password_reset_confirm  	| PasswordResetConfirmView  	| SetPasswordForm    	| password_reset_confirm.html  	|
| password_reset_complete 	| PasswordResetCompleteView 	|                    	| password_reset_complete.html 	|
|                         	|                           	| UserCreationForm   	|                              	|
|                         	|                           	| UserChangeForm     	|                              	|

`PasswordResetView` also uses `password_reset_email.html` and `password_reset_subject.txt`.

Settings to use:
* `LOGIN_URL`: url of login page. Used by many parts of Django
* `LOGIN_REDIRECT_URL`: where redirects after login. Used by `LoginView`.
* `LOGOUT_REDIRECT_URL`: where redirects after logout. Used by `LogoutView`.
* `DEFAULT_FROM_MAIL`: Email sender email. Used by `PasswordResetView`.
* `PASSWORD_RESET_TIMEOUT`: In seconds. Time until you can reset the password since the email has been sent.

### Customizing auth

#### Extending User model. 2 options:

1 - Custom user model that extends `django.contrib.auth.models.AbstractUser`.
* Always create it BEFORE the first migration. Fix it after first migrations is impossible.
* If your custom user model has `is_active`, only users with `is_active = True` can be authenticated.
* In settings: `AUTH_USER_MODEL` must point to your new user model
* In admin: `admin.site.register(NewCustomUser, django.contrib.auth.admin.UserAdmin)`
* When refering it, use: `User = auth.get_user_model()`
  * Except in models and signals, where you must use: `settings.AUTH_USER_MODEL`
* Although it is not common and not recommended, if your custom user model changes the basic fields of `AbstractUser`, you must do too:
  * Create a specific `UserManager`
  * Specify `USERNAME_FIELD`
  * Change `UserCreationForm` and `UserChangeForm`
  * Register class in admin, withour using `UserAdmin`

2 - OneToOneField pointing to the user model
* E.g.: model class Profile, with: `user = models.OneToOneField(User, on_delete=models.CASCADE)`
* `user.profile` for using it


#### Custom auth backend

In settings, add AUTHENTICATION_BACKENDS = [......]
> `[django..contrib.auth.backends.ModelBackend,` <- Default backend
>  `myproject.app1-backendsAuth.MyBackend]`
Create backend class that inherits from BaseBackend with 2 methods:
```python
class MyBackend(BaseBackend):
    authenticate(self, request, username, password) -> User
    get_user(user_id) -> User
```
`user_id` must be primary key


## AUTHORIZATION - PERMISSIONS

### Built-in permissions system (Model level permissions)

Best doc:[Permissions](https://testdriven.io/blog/django-permissions/)

For every model there are 4 types of permissions/actions: view, add, change, delete. Permissions can be set to a user o to a group. To check if a user has certain permission, you can use the ModelAdmin or the own user.

| **ModelAdmin**          	| **User**                        	|
|-------------------------	|---------------------------------	|
| has_view_permission()   	| user.has_perm('foo.view_bar')   	|
| has_add_permission()    	| user.has_perm('foo.add_bar')    	|
| has_change_permission() 	| user.has_perm('foo.change_bar') 	|
| has_delete_permission() 	| user.has_perm('foo.delete_bar') 	|

Permission naming sequence: {app}.{action}_{model_name}

To modify permissions:
myuser.groups.set([group_list])
myuser.groups.add(group, group, ...)
myuser.groups.remove(group, group, ...)
myuser.groups.clear()
myuser.user_permissions.set([permission_list])
myuser.user_permissions.add(permission, permission, ...)
myuser.user_permissions.remove(permission, permission, ...)
myuser.user_permissions.clear()


### Decorators and Mixins

Decorators for function based views. Mixins models for class based views.

| **decorators**           	| **mixins classes**      	|                               	|
|--------------------------	|-------------------------	|-------------------------------	|
| `@login_required`        	| LoginRequiredMixin      	| (False) -> settings.LOGIN_URL 	|
| `@staff_member_required` 	|                         	| For only staff and admins     	|
| `@user_passes_test`      	| UserPassesTestMixin     	| Custom test authorization     	|
| @permission_required     	| PermissionRequiredMixin 	| Built-in django permissions   	|

To use a mixin, simply make your view inherits the mixin class (the firist in the parents list): `class MyView(LoginRequiredMixin, View)`. `raise_exception` must be `True` to raise a 403 HTTP Forbidden error.

To use a decorator simply place it before the function view:
```python
@loguin_required
def my_view(request):
```

When your view inherits from `UserPassesTestMixin`, it must override `test_func()`, where you check if the user is authorized to that request, and return a boolean.
```python
class MyView(UserPassesTestMixin, UpdateView):
    def test_func(self) -> bool:
      ......
```

To use `user_passes_test` pass it callables as arguments. Those callabes must be functions that takes an user as argument and return a boolean.
```python
def my_test_func(user) -> bool:
  .....

@user_passes_test(my_test_func)
def my_view(request):
```

@permission_required example:
```python
@permission_required('polls.add_choice', raise_exception=True)
def my_view(request):
    ...
```

`PermissionRequiredMixin` example:
```python
class MyView(PermissionRequiredMixin, View):
    permission_required = 'polls.add_choice'
    # Or multiple of permissions:
    permission_required = ('polls.view_choice', 'polls.change_choice')
```

### Django Authorization Best Practices

Best doc: [Custom authorization](https://learndjango.com/tutorials/django-best-practices-user-permissions)

#### In class-based views:
* Inherits `LoginRequiredMixin`, `UserPassesTestMixin` in that order
* Override `test_func()`. Within it use `self.request.user` to get the user and `self.get_object()` to get the model instance in that view.
* (optional) Create authorization.py and place the functions that ckeck the permission there. Those functions must return a boolenan. Call them from `test_func()`.
* Field `raise_exception = True`.

```python
class BlogUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    template_name = 'post_edit.html'
    fields = ['title', 'body']
    raise_exception = True
    def test_func(self):
        obj = self.get_object()
        return obj.author == self.request.user
```

#### In function based views:
* @loguin_required
* Create authorization.py and place the functions for checking permissions there. Those functions must raise a `PermissionDenied` or a `HttpResponseForbidden` if the user isn't allowed for that request.
* Call functions in authorization.py after getting the objects. 

```python
#views.py
@loguin_required
def my_view(request, my_model_id):
  my_model_object = get_object_or_404(MyModel, pk=my_model_id)
  authorization.check_permissions_my_model_my_view(my_model, request.user)

#authorization.py
def check_permissions_my_model_my_view(my_model:MyModel, user:User):
  if ....:
    raise PermissionDenied()
  else:
    ....
```


## TEST

### Commands

test my_app
test my_app.tests_views
test my_app.tests.test_views (if you have a 'tests' folder)
test my_app.test_views.MyViewTestCase


## FILES AND IMAGES

### Uploading files by users

First of all you must:
* If you are going to store images, install pillow: `pip install pillow`
* To make your project able to work with files in DEBUG mode, you must add this code at the end of urls.py general file:
```python
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
Settings must have:
* `MEDIA_ROOT`: Absolute path to the directory that will hold the files 
  * It is common:`MEDIA_ROOT = os.path.join(BASE_DIR, 'media')`. For development at least.
* `MEDIA_URL`: URL that serves the files stored at this location

In models.py:
* Files: `models.FileField(upload_to='files/')`
* Images: `models.ImageField(upload_to='images/')`
* `upload_to` will be the folder within `MEDIA_ROOT` for storing the files.

In templates/HTML:
* For forms to upload file: modify ´<form>´ to: `<form enctype="multipart/form-data">`
* To show an image: `<img src="{{profile.images.url}}">`

In forms.py:
* If ModelForm: Just add the field to Meta class
* If Form:
  * Files: `forms.FileField(... widget=forms.FileInput(attr={..})...)`
  * Images: `forms.ImageField(... widget=forms.FileInput(attr={..})...)`
  * Keep in mind that `ImageField`'s widget is `FileInput`

In views.py:
* Add `files=request.FILES` when creating a Form
  *e.g.: `MyForm(data=request.POST, files=request.FILES, ...)`

### Managing files

Not started.... the File class

## FORMS

A way to organize fields, labels and widgets:
```python
MyForm(Form): 
    class Meta:
        fields=('...', '...', '...')
        labels={'...', '...', '...'}
        widgets={'...':forms.TextInput(attrs={'..':'..', ..}), ....}
```

## VIEWS

Class based views:

Order methods:
1 setup() -> 2 dispatch() -> 3 get()
                          -> 3 post()

## PACKAGES TO USE

### django-axes: for preventing brute force attacks at login

### django-crispy-forms: for rendering 

## See also

