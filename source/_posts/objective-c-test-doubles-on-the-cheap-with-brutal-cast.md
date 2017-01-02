---
title: objective-c test doubles on the cheap with brutal cast
id: 113
date: 2010-09-13 16:56:39
tags:
- brutal cast
- casting
- dispatch
- dynamic
- interactionist
- message
- objective-c
- test doubles
- testing
- unit tests
---

Objective-C has the power of Ruby, with duck-typing and dynamic dispatch in the object layer. At the same time it has the power of C, with direct access to memory layouts and static-weak typing below the object layer. Sometimes, the two powers can be combined for some unexpected results.

On my current project we are trying to unit test as much functionality as we reasonably can. I am quite happy to write interactionist tests, so I need test doubles. Although the Objective-C compiler does static type checking at compile time, at run-time Objective-C objects will respond to any message for which they have a method defined.

This makes creating test doubles very easy. Consider a controller that accepts an error delegated from a CLLocationManager, and delegates it on to a logging class. Fragments of the classes involved might look like this:
<pre lang="objective-c">@interface Logger : NSObject

- (void)log:(NSError *)error;

@end

@interface LocationSensitiveController <CLLocationManagerDelegate> : NSObject

- (id)initWithLogger:(Logger *)logger;
- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error;

@end</pre>

In my test I would like to use a test double in place of the logger, and assert that the same error gets passed along:
<pre lang="objective-c">- (void)testShouldPassErrorToLogger;
{
  Logger *stubLogger = // how to create the stub logger?
  LocationSensitiveController *controller = [[[LocationSensitiveController alloc] initWithLogger:stubLogger] autorelease];
</pre>

The stub logger need only understand the log: message to serve its purpose. It does not need to have any relationship to the Logger class. I've been calling these classes "Pretend..." because the class is only pretending to be the other type. They would be stubs in the [Test Double taxonomy](http://martinfowler.com/articles/mocksArentStubs.html#TheDifferenceBetweenMocksAndStubs) that Martin Fowler popularised.
<pre lang="objective-c">@interface PretendLogger : NSObject

- (void)log:(NSError *)error;
- (NSError *)receivedError;

@end</pre>

The compiler will reject a straight assignment:
<pre lang="objective-c">  Logger *stubLogger = [[[PretendLogger alloc] init] autorelease]; // type error</pre>

The low-level C power can be used to convince the compiler otherwise:
<pre lang="objective-c">  Logger *stubLogger = (Logger *)[[[PretendLogger alloc] init] autorelease];</pre>

In C this type of cast is sometimes called a brutal cast. The cast tells the compiler to interpret the same area of memory as a different type. All Objective-C classes share the same basic memory layout, so in the example the cast "sneaks" the PretendLogger past the compile-time static checking and in to the LocationSensitiveController instance. There it will receive messages intended for Logger, and because it implements a method for the same selector (log:), the code will run successfully.

Using a cast, I can write the test using my PretendLogger class:
<pre lang="objective-c">@implementation LocationSensitiveControllerTest

- (void)testShouldPassErrorToLogger;
{
  Logger *stubLogger = [[[PretendLogger alloc] init] autorelease];
  LocationSensitiveController *controller = [[[LocationSensitiveController alloc] initWithLogger:stubLogger] autorelease];
  NSError *expectedError = [NSError errorWithDomain:@"domain string" code:666 userInfo:nil];

  [controller locationManager:nil didFailWithError:expectedError];

  NSError *actualError = [(PretendLogger *)stubLogger receivedError];
  GHAssertEquals(expectedError, actualError, @"error should be received by logger");
}

@end</pre>

Eventually a [mocking framework](http://www.mulle-kybernetik.com/software/OCMock/) makes sense, or real classes can be used with [method swizzling](http://www.cocoadev.com/index.pl?MethodSwizzling). When getting started on a project or a new area of code, this is a very simple approach to get some interaction tests going.

I've posted a complete xcode project incorporating the example test to [github](http://github.com/davcamer/cheap_objc_mocks).
