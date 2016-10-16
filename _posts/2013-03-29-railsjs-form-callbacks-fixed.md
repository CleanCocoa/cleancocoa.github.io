---
title: Rails.js XHR form callbacks fixed for invalid submissions
created_at: 2013-03-29 11:00 +0100
updated_at: 2013-04-14 11:06 +0200
kind: worklog
tags: [rubyonrails, jquery, javascript, ajax]
---

tl;dr
:   Ajax form callbacks not executing your `js.erb` template contents when validation returned errors? [There's a solution to that,][2] utilizing `ajax:error` callbacks for server responses that indicate client-side errors.

I struggled with Ajax form calls in Ruby on Rails for some time now.  Because I seem to be anal retentive, I insist the server should respond with HTTP status code "400 Bad Request" or "424 Unprocessable Entity" when form submission turns out to be full with invalid data.

Responding with "200&nbsp;OK" doesn't make any sense when there was an error on the client side, like an empty or otherwise invalid form.  That's what HTTP codes are for:  to comminicate how it went and what happened.  Also, we web developers are creating publicly visible Web Services and APIs here, so they should conform to the most basic standards of the transfer protocol we're using, am I right?

Well, Rails renders and responds everything like expected, but the JavaScript code won't be executed client side after invalid form submission.  Valid form submissions work fine all the time, by the way.

- **Expected Behavior:** Rails renders JavaScript code templates on the server side, sends it in the response body, so the client can execute the JavaScript code when the request is done.
- **Real-world Behavior:** Rails renders JavaScript code templates and responds as it should, but no code will be executed on the client.

Because of that, I resolved to reproduce the behavior and rendering on the client side, responding with raw message data instead.  Thing was, I could very well _read_ the response on client side, so it only felt natural to scoop the raw error messages through.

Take a look at the CoffeeScript, Unobstrusive JavaScript and HTML rendering monster I created; the important [code files are in a GitHub Gist][3] to unclutter this post.

Of course my problems would have simply evaporated in an instant, had I changed the status code to, say, "200&nbsp;OK".  Wasn't an option.

[3]: https://gist.github.com/DivineDominion/5269802

## Make rails.js, the Unobstrusive JavaScript/jQuery driver execute form callbacks

But in the end it turns out you could simply execute the Ajax/XMLHttpRequest (XHR) response body via `eval()`.  Here's some JavaScript to do exactly that in case the server's response isn't, like, "200&nbsp;OK" and will translate to an `ajax:success` callback but triggers `ajax:error` instead, on any form whatsoever:

    // https://gist.github.com/DivineDominion/5269849
    $(document).ready(function() {
      $(document).on('ajax:error', 'form[data-remote="true"]', function(evt, xhr, status) {
        eval(xhr.responseText);
      });
    });

  [stackov]: http://stackoverflow.com/questions/203198/event-binding-on-dynamically-created-elements

Update
: Binds the event on `$(document)` so that [dynamically created forms will be affected, too.][stackov]

## Full sample Rails project with working form validation

I really struggled with this for some time, since Steve Schwartz aka JangoSteve says Rails' JavaScript responses should be executed no matter what.  There's no condition like "only when it's a successful response"---I mean, look, even [Steve's examples][1] respond with status code "424 Unprocessable Entity".  The fault clearly must be on my side, somewhere in my code, must it not?

After I created a plain vanilla Rails project based on his example code snippets, I confirmed the seemingly faulty behavior and found I did nothing wrong.  What relief!

I added the abovementioned general `ajax:error` callback to fix this for anyones reference and convenience.

**Go and [get the working Rails project from GitHub][2] and try it for yourself.**

[Tell me][4] if it was useful!

[1]: http://www.alfajango.com/blog/rails-3-remote-links-and-forms-data-type-with-jquery/
[2]: https://github.com/DivineDominion/rails.js_form_response
[4]: http://twitter.com/ctietze
