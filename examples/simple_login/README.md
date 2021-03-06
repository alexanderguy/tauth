# simple_login tauth example

This example demonstrates tauth wrapping a simple service, with django
providing the backend for authentication and authorization.

	- nginx.conf - specifies the exposed services
	- nginx.lua - additional nginx-specific lua configuration
	  and binding of the django auth API into tauth.

Using the simple_login example
------------------------------

## 0. Moving Parts

This example shows how to use the built-in Django authentication system to
authorize for tauth.  To use it you'll need:

	- django (e.g. 1.6.x).
	- A recent nginx built with lua/luajit (e.g. 1.6.x).
	- uwsgi (e.g. 2.0.x).
	- lua-resty-http

All of the above should be available in a relatively fresh distribution (e.g. Debian/testing).

## 1. Setup django state:

	cd tauth/examples/simple_login
	./manage.py collectstatic
	./manage.py syncdb

## 2. Run django in the foreground under uwsgi:

	uwsgi --ini uwsgi.ini

## 3. In another shell, run nginx in the foreground:

	/usr/sbin/nginx -p . -c nginx.conf

## 4. Attempt to access the protected time service e.g.:

	http://localhost:8080/time

You should see an uwsgi access, and your browser should show a '401
Authorization Required' message.

## 5. Log in to django using your browser, and the account you created in step #1 e.g.:

	http://localhost:8080/accounts/login

## 6. Attempt to go to the protected time service e.g.:

	http://localhost:8080/time

You should see another 401 message.

## 7. Navigate to the django admin page, and add a tauth role to your user.

You can do this by navigating to the 'Change user' page, scrolling to the bottom,
and clicking the green plus near 'Role:'.  Note: the value of the role URI doesn't currently
matter, it just needs to have a value.  Save the user.

## 8. Create some resources and permissions for the examples.

Add the following resources:

	tauth:simple_login:resources:echo
	tauth:simple_login:resources:time

Add two permissions for the role you created, with the resources above.  Make them both 'ALLOW'
with an action_uri of 'tauth:simple_login:actions:access'.

## 9. Attempt to go to the protected time service e.g.:

	http://localhost:8080/time

You should see the current UTC time presented to you.

## 10. Log out of django, e.g. :

	http://localhost:8080/accounts/logout

## 11. Attempt to go to the protected time service e.g.:

	http://localhost:8080/time

You should see the 401 message again.

