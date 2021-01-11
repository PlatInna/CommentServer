# Web Server of Comments
Coursera: C++ Development Fundamentals: Brown Belt: Week 2

Condition

Imagine - You have joined the team that is developing the comment web server. This server allows you to create new users, post new ads, and read all the ads of the selected user. 
In addition, the team has recently attended to the fight against spammers. If a user is recognized as a spammer, he is blocked, after which he is given a captcha page, 
on which he must confirm that he is a human. If the captcha is successfully entered, the user will be unblocked and will be able to post comments again.

To identify spammers, a fairly simple algorithm is used - a user who has sent three comments in a row is recognized as a spammer.

The server works over the HTTP protocol and processes the following requests:
- ```POST /add_user``` - adds a new user and returns a ```200 OK``` response, the body of which contains the ID of the newly added user (see the implementation in the solution stub)
- ```POST /add_comment``` - extracts user ID and new comment from the request body; if the user is recognized as a spammer, returns ```302 Found``` with the header 
```Location: /captcha```, transferring the user to the captcha page, otherwise saves the comment and returns ```200 OK```
- ```GET /user_comments? User_id = [id]``` - returns a ```200 OK``` response, the body of which contains all the comments of the user id, separated by newlines
- ```GET /captcha``` - returns a ```200 OK``` response, the body of which contains a captcha page (for simplicity, in this task, we simply return a question to be answered 
by the user, in practice it can be a full-fledged HTML page)
- ```POST /checkcaptcha``` - extracts the answer to the captcha question from the request body; if it is correct, it unblocks the user and returns ```200 OK```, if not, 
returns ```302 Found``` with the header ```Location: /captcha```
- if the request method is not ```POST``` or ```GET```, or the request path does not match any of the above, the server responds with ```404 Not found```.

The web server in code is implemented using the class ```CommentServer```:

```c++
struct HttpRequest {
  string method, path, body;
  map<string, string> get_params;
};

class CommentServer {
public:
  void ServeRequest(const HttpRequest& req, ostream& response_output);

private:
  ...
};
```
Its ```ServeRequest``` method accepts the HTTP request, processes it, and writes the HTTP response to the ```response_output``` output stream (this stream can be bound 
to a network connection). When writing the HTTP response to the output stream, the following format is used:

```
HTTP/1.1 [response code] [comment]
[Header 1]: [Header value 1]
[Header 2]: [Header value 2]
...
[Header N]: [Header value N]
<empty line>
[Response body]
```
- Response code is 200, 302, 404, etc.
- Comment - "Found", "OK", "Not found", etc.
- Header X - Header name, eg "Location"
- Response body for example, this is the content of the captcha page or the identifier of a newly added user; however, if the response body is non-empty, the response must contain the ```Content-Length``` header, the value of which is equal to the response length in bytes.
