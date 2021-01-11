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
- Response body - for example, this is the content of the captcha page or the identifier of a newly added user; however, if the response body is non-empty, the response must contain the ```Content-Length``` header, the value of which is equal to the response length in bytes.

An example of response to the ```/add_user``` request that returns a new user ID of 12. Content-Length is 2 because "12" is two characters:
```
HTTP/1.1 200 OK
Content-Length: 2

12
```
There is a problem with our server - sometimes it does not respond to requests, and sometimes it returns incorrectly formed responses. The source of these problems is that the answers are generated manually each time. Because of this, we either forgot the line feed, then added an extra one, then we sealed ourselves in the response code:
```c++
void ServeRequest(const HttpRequest& req, ostream& os) {
  if (req.method == "POST") {
    if (req.path == "/add_user") {
      comments_.emplace_back();
      auto response = to_string(comments_.size() - 1);
      os << "HTTP/1.1 200 OK\n" << "Content-Length: " << response.size() << "\n" << "\n"
        << response;
    } else if (req.path == "/checkcaptcha") {
       ...
        os << "HTTP/1.1  20 OK\n\n";
      }
    } else {
      os << "HTTP/1.1 404 Not found\n\n";
    }
  ...
}
```
You decided to get rid of all the problems at once and carry out the following refactoring:
- develop an ```HttpResponse``` class that will represent the HTTP-response; in operator ```<<``` you have decided to encapsulate the output format of the HTTP-response into a stream
- make a new signature of the ```ServeRequest``` method - ```HttpResponse ServeRequest (const HttpRequest & req)``` - which at the compilation stage will ensure that our server always returns at least some response (if we forget to do this, the compiler will issue a warning "control reaches end of non-void function ")
- write the server response to the output stream in one single place where the ```ServeRequest``` method is called
You decided to make the interface of the ```HttpResponse``` class like this:
```c++
enum class HttpCode {
  Ok = 200,
  NotFound = 404,
  Found = 302,
};

class HttpResponse {
public:
  explicit HttpResponse(HttpCode code);

  HttpResponse& AddHeader(string name, string value);
  HttpResponse& SetContent(string a_content);
  HttpResponse& SetCode(HttpCode a_code);

  friend ostream& operator << (ostream& output, const HttpResponse& resp);
};
```
The ```AddHeader```, ```SetContent``` and ```SetCode``` methods must return a reference to themselves in order to be able to form a response in one line using chaining: 
```return HttpResponse(HttpCode::Found).AddHeader("Location", "/ captcha");```. 
The ```HttpCode``` enumeration passed to the constructor of the ```HttpResponse``` class ensures that we don't make a mistake in the response code.

This refactoring is what you will be doing in this task. The server must implement the protocol described above.
Clarifications to the implementation of the HttpResponse class:
- The ```AddHeader```, ```SetContent```, ```SetCode``` methods must return a reference to the object for which they are called
- The ```AddHeader``` method always adds a new header to the response, even if there is already a header with the same name
- operator ```<<``` for the ```HttpResponse``` class should output an HTTP-response in the format described above in the description of the ```ServerRequest``` method; in this case, the headers can be displayed in any order. If the HTTP-response has non-empty content, then it is necessary to print exactly one "Content-Length" header (in addition to the headers contained in the HTTP-response), the value of which is equal to the size of the content in bytes.
