
[title(emptyline_above_is_important)]: # (Django: Decorators for Permissions Checks)

# Role-Based Access Using Decorators
Below, I go through a Python decorator which takes arguments to enforce a granular level of permission handling on Django REST Framework API Views.

## Why use a decorator instead of a custom middleware?
Implementing this feature as a middleware would apply globally i.e., every view used by the app would be affected.
I needed to granularly enforce the user permissions on a per-view basis, as not all views needed **Authentication**.
There would be no point in checking the permissions of the role of the **AnonymousUser**.
Case in point, we wouldn't require endpoints used by the public-facing landing page to require visitors be logged in.

Additionally, each view authorizes a different set of roles.
For example, endpoints for data dashboards might require more stringent write (`POST`, `PUT`, `PATCH`, `DELETE`)
permissions given only to administrators or managers. In contrast, read (`GET`) permissions might be given to all support staff.


## Writing the Decorator
```python
# examples/decorators.py

from functools import wraps
from rest_framework import status
from rest_framework.response import Response

def permit_if_role_in(allowed_roles=()):
    """ This decorator takes arguments and returns a Closure that utilizes these arguments. """

    def view_wrapper_function(decorated_view_function):
        """ This intermediate wrapper function takes the decorated View function (e.g. get, post) itself. """
    
        @wraps(decorated_view_function)
        def enforce_user_permissions(view, request, *args, **kwargs):
            """ A function that intercepts the View function and enforces permissions """
        
            # Perform permissions checking here, before control is passed to the View function that was decorated
            permissions_evaluations = map(lambda role: gettattr(user, role, False), allowed_roles)
            is_authorized = any(permissions_evaluations)
    
            if not is_authorized:
                return Response("You are authenticated but unauthorized!", status=status.HTTP_403_FORBIDDEN)
        

            # Passing the arguments and control over to the view function that was decorated
            response =  decorated_view_function(view, request, *args, **kwargs)

            # Perform actions here after the control is returned by the view function that was decorated
            
            return response
    
        return enforce_user_permissions
    
    return view_wrapper_function
```

## Using the Decorator
Below I show some examples of how to use the decorator in custom *View*s and *ViewSet*s

### APIView example
```python
# examples/views.py

from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from .decorators import permit_if_role_in

class MyDecoratedAPIView(APIView):
    """ A sample APIView to demonstrate the decorator usage """

    permission_classes = [IsAuthenticated]

    @permit_if_role_in(['is_support', 'is_staff'])
    def get(self, request):
        """ This function only gets called if request.user.is_support or request.user.is_staff is True """

        # Your get code here
        pass

    @permit_if_role_in(['is_admin', 'is_management'])
    def post(self, request):
        """ This function only gets called if request.user.is_admin or request.user.is_management is True """

        # Your get code here
        pass
```

### ViewSet example
```python
# examples/views.py

from rest_framework.permissions import IsAuthenticated
from rest_framework.viewsets import ViewSet
from .decorators import permit_if_role_in

class MyDecoratedViewSet(ViewSet):

    permission_classes = [IsAuthenticated]

    @permit_if_role_in(['is_support', 'is_staff'])
    def fetch_dashboard_data(self, request):
        """ This function only gets called if request.user.is_support or request.user.is_staff is True """

        # Your get code here
        pass

    @permit_if_role_in(['is_admin', 'is_management'])
    def modify_dashboard_data(self, request):
        """ This function only gets called if request.user.is_admin or request.user.is_management is True """

        # Your get code here
        pass
```


## How and Why Does This Work?
By decorating `@permit_if_role_in(allowed_roles)` to the `fetch_dashboard_data(self, request)` function,
we are actually performing the following equivalent statement, which is a series of nested function calls.
```python
response = permit_if_role_in(allowed_roles)(modify_dashboard_data)(request, *args, **kwargs)
```

Let's break down the convoluted statement above into shorter, more manageabl statements using intermediate variables.
```python
allowed_roles = ['is_admin', 'is_management']

# `permit_if_role_in` takes a list/tuple of `allowed_roles` and returns a callable `view_wrapper_function`
view_wrapper_function = permit_if_role_in(allowed_roles)

# `view_wrapper_function` takes an APIView `modify_dashboad_data` to wrap over
# and returns its own APIView function `enforce_user_permissions`
enforce_user_permissions = view_wrapper_function(modify_dashboad_data)

# `enforce_user_permissions` takes the rest_framework.request.Request() passed to the view,
# along with other optional args and kwargs
response = enforce_user_permissions(request, *args, **kwargs)
```

And finally, it is inside `enforce_user_permissions` where the checking of `allowed_roles` is done, before passing
control to `modify_dashboad_data`.

With just a single line modification, so much takes place under the hood.
That is the power syntactic sugar Python provides.


## Closing
So I've shown you how I used these decorators and closures to implement a per-view permissions checker for your views.
Functional programming is a tricky subject to learn at first, but it's important to remember that in Python, functions
(or callables) are also objects that can be passed around as if they were primitive data types. I hope the examples
helped.