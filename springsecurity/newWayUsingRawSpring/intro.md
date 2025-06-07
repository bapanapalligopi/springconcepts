spring security
=======================
# Normal Flow
Client->Request -> FrontController(Dispatcher Servlet)->list of controllers in our application.

![image](https://github.com/user-attachments/assets/b7bf038c-723c-4974-b3dc-ff3ebfa625cd)

# in Security
Client -> Request-> Request needs to be passed through the Filter(List Of Filters)->list of controllers with endpoint

Filters can handle whether the request is authorized or forbidden.
Resources are protected by filters that filter incoming requests from the client.
![image](https://github.com/user-attachments/assets/431b3a58-e29e-499e-b79e-4af6ba2350c1)

we do not need to setup filters just we can use spring security framewrok and use provided classes.
