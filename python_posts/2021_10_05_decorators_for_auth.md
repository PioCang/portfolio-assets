
[title(emptyline_above_is_important)]: # (Django: decorators)

# Role-Based Access Using Decorators
Below, go I through using Python decorators -- which take arguments -- to enforce a granular level of permission handling on Django REST Framework API Views.

# Why use a decorator instead of a custom middleware?
Implementing this feature as a middleware would apply globally i.e., every view used by the app would be affected. I wanted to granularly enforce the user permissions on a per-view basis, as not all views needed **Authentication**. There would be no point in checking the permissions of the role of the **AnonymousUser**. Case in point, we wouldn't require endpoints used by the public-facing landing page to require visitors be logged in.

Additionally, each view authorizes a different set of roles. For example, endpoints for data dashboards might require more stringent write permissions given only to administrators or managers, while read permissions are given to all support staff.



```python
from functools import wraps
from rest_framework import status
from rest_framework.response import Response

def permit_if_role_in(allowed_roles=()):
    """
    This decorator takes arguments. Decorate the API View like so:
    
    @permit_if_role_in(('is_staff', 'is_admin'))
    """

    def view_wrapper_function(api_view_function):
        """ This intermediate wrapper function doesn't take arguments """
    
        @wraps(api_view_function)
        def enforce_user_permissions(view, request, *args, **kwargs):
            """ A function that intercepts a View function and enforces permissions """
        
            # Perform permissions checking here, before control is passed to the View function that was decorated
            permissions_evaluations = map(lambda role: gettattr(user, role, False), allowed_roles)
            is_authorized = any(permissions_evaluations)
    
            if not is_authorized:
                return Response("You are authenticated but unauthorized!", status=status.HTTP_403_FORBIDDEN)
        

            # Passing the arguments and control over to the view function that was decorated
            response =  api_view_function(view, request, *args, **kwargs)

            
            # Perform actions here after the control is returned by the view function that was decorated
            
            return response
    
        return enforce_user_permissions
    
    return view_wrapper_function
```


```
```