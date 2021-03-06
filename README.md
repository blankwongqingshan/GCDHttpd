GCDHttpd
========

GCDHttpd is a lightweight objective-c HTTP server framework build atop
[GCDAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket),
thus it is easy to be embeded in iOS/OSX projects.  The underline
grand central dispatch make the multi core programming easy.

## Install

* Drag the Folder GCDHttpd into your Xcode projects
* Add Security.framework and CFNetwork.framework into your Xcode projects

## Example Usage

```
    // Initialize the httpd
    httpd = [[GCDHttpd alloc] initWithDispatchQueue:dispatch_get_current_queue()];
    httpd.port = 8000;       // Listen on 0.0.0.0:8000
    // Router setup
    [httpd addTarget:self action:@selector(deferredIndex:) forMethod:@"GET" role:@"/users/:userid"];
    [httpd addTarget:self action:@selector(simpleIndex:) forMethod:@"GET" role:@"/"];
    [httpd serveDirectory:@"/tmp/" forURLPrefix:@"/t/"];    // Static file serving "/t/"
    [httpd serveResource:@"screen.png" forRole:@"/screen.png"];   // Resource in the main bundle

    [httpd start];
...

- (id)simpleIndex:(GCDRequest *)request {
    return @"hello";
}

- (id)deferredIndex:(GCDRequest*)request {
    NSString * message = [NSString stringWithFormat:@"hello %@", request.pathBindings[@"userid"]];
    GCDResponse * response = [request responseWithContentLength:message.length];
    response.deferred = YES;
    
    // This request lasts 2 seconds
    double delayInSeconds = 2.0;
    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayInSeconds * NSEC_PER_SEC));
    dispatch_after(popTime, dispatch_get_current_queue(), ^(void){
        [response sendString:message];  // Send message 2 seconds later
        [response finish];    // Finish the response
    });
    return response;
}

```


