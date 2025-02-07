---
title: Good mulle-objc Code
keywords: class
last_updated: March 26, 2019
tags: [language]
summary: "A functional `.m` file that contains all the features that are
available in mulle-objc."
permalink: mydoc_good.html
folder: mydoc
---

## Good Code

Objective-C is a fairly simple language extension. In this file all available
Objective-C keywords and concepts will be discussed. Absent will be library
features introduced by the Foundation.

The code will be regular Objective-C, unless specifically noted to be an
Objective-C extension.


### Forward Declaration and Import

You can `#import`  and forward declare code:

``` objc
#import <Foundation/Foundation.h>

// forward declarations of a class and a protocol
@protocol SomeProtocol;
@class Foreign;
```

### Protocol

You can declare a protocol with **@optional**  and **@required** methods and
properties:

``` objc
// declare a protocol with a required (default)
// and an optional method
@protocol MethodProtocol

@required
- (Foreign *) someMethod;

@optional
- (Foreign *) otherMethod;

// properties can be declared in protocols, yet the
// class must redeclare them

@property( assign) NSUInteger  value;

@end
```

You can extend forward declared classes with your protocol, but this is rarely
useful:

``` objc
// extend Foreign class with this method, so
// we know we can message it with that protocol
@class Foreign< MethodProtocol>;
```

### Classes

``` objc
#pragma clang diagnostic ignored "-Wobjc-root-class"
@interface Foo : NSObject < MethodProtocol>
{
@public        // The usual visibility modifiers except `@package`
@protected
@private
   NSUInteger   _value;    // will be used as property "value" ivar
}

// properties and all supported property attributes
@property( readwrite, assign) NSUInteger  value;      // reimplement Protocol
@property( nonatomic, retain) Foreign     *foreign;   // creates own ivar
@property( readonly, nonnull) NSString    *backedByIvar; // creates own ivar

@end
```

The implementation can **@synthesize** a property but it is superflous. The name
of the instance variable is fixed by the compiler to be `'_'<name>`.
`Foo` implements the required method from `MethodProtocol`.

``` objc
@implementation Foo

@synthesize foreign = _foreign;

- (Foreign *) someMethod;
{
   return( _foreign);
}

@end
```

**@defs** is available and great for writing fast accessors, when you don't want
your instance variables to be **@public**:

``` objc
static inline Foreign  *FooGetForeign( Foo *self)
{
   return( ((struct{ @defs( Foo); } *) self)->_foreign);
}

```

#### @compatibility_alias

**@compatibility_alias** will work as expected.

``` objc
// use Foo under alias Foobar
@compatibility_alias Foobar Foo;
```


### Protocolclasses

You can declare [protocolclasses](mydoc_protocolclass.html). This is a
mulle-objc extension to the Objective-C language. Protocolclasses will not work
on other runtimes (though they will compile):

``` objc
// protocolclass Description with default implementation of -description
@class Description;
@protocol Description
- (NSString *) description;

// properties can not be declared in protocolclases yet
// @property( assign) NSUInteger  value;
@end
#pragma clang diagnostic ignored "-Wobjc-root-class"

// protocolclass must implement Protocol of same name
@interface Description< Description>
@end

// implementation will serve the default implementation of description
@implementation Description

- (NSString *) description
{
   return( @"VfL Bochum 1848");
}
@end
```

Your classes can now inherit the description protocol, but it doesn't have to
implement `-description`. It can override it though if desired:

``` objc
@interface Bar : Foo < Description>
@end


@implementation Bar

// call super on protocolclass (not superclass)
- (NSString *) description
{
   return( [super description]);  // unvoidable warning for now
}
```

### Keywords


#### PROTOCOL

**PROTOCOL** is a compiler keyword. Objective-C's `Protocol *` does not work, as
PROTOCOL is a kind of **@selector** in mulle-objc:


``` objc
- (BOOL) knowsThisProtocol:(PROTOCOL) proto
{
   return( @protocol( Description) == proto);
}
```

#### instancetype

**instancetype** is a standin for `id` the generic object pointer:


``` objc
// instancetype keyword
- (instancetype) init
{
   return( self);
}
```

#### @encode

**@encode** behaves normally and is 90% compatible with the Apple runtime:

``` objc
// @encode keyword
+ (char *) type
{
   return( @encode( Bar));
}
```

#### @autoreleasepool

You can use **@autoreleasepool** to create nested pool structures:

``` objc
// keyword autoreleasepool
- (void) autoreleasepool
{
   @autoreleasepool
   {
   }
}
```

#### @try ...

Exception handling can be done with **@try**,**@catch**,**@finally**:

``` objc
- (void) trycatchfinally
{
   @try
   {
   }
   @catch( NSException *e)
   {
   }
   @finally
   {
   }
}
```

#### NS_DURING ...

Or with old-skool  **`NS_DURING`**,**`NS_HANDLER`**,**`NS_ENDHANDLER`**:

``` objc
// exception handling with nsduring
- (void) nsduring
{
NS_DURING
NS_HANDLER
   [localException raise];
NS_ENDHANDLER
}
```


### Literals

Literals are supported *except* **@YES** and **@NO**, but boxed versions of those are fine.

#### @1848,@18.48,@'A',@(YES)

``` objc
- (NSNumber *) literalInteger
{
   return( @1848);
}


- (NSNumber *) literalDouble
{
   return( @18.48);
}


- (NSNumber *) literalCharacter
{
   return( @'A');
}

- (NSNumber *) literalBOOL
{
   return( @(YES));
}
```

#### @{} and @[]

``` objc
- (NSDictionary *) literalDictionary
{
   return( @{ @"VfL Bochum" : @1848 });
}

- (NSArray *) literalArray
{
   return( @[ @1848, @"VfL Bochum" ]);
}
```

#### @protocol

``` objc
- (PROTOCOL) literalProtocol
{
   return( @protocol( Description));
}
```

#### @selector

``` objc
- (SEL) literalSelector
{
   return( @selector( whatever:));
}
```


### Fast Enumeration

*Fast Enumeration* is supported:

``` objc
// fast enumeration
- (void) fastEnumerate:(NSArray *) array
{
   for( id p in array)
   {
   }
}

@end
```

> Checked against the list of [NSHipster's compiler directive](https://nshipster.com/at-compiler-directives/)

## Putting all the pieces together

Run the `main` function, to verify the truth of what's been written ([good-code.m]({{ site.baseurl}}/files/good-code.m)).


``` objc
int main()
{
   Bar   *bar;

   bar = [Bar object];

   [bar fastEnumerate:[NSArray arrayWithObject:@"foo"]];

   mulle_printf( "description       : %@", [bar description]);
   mulle_printf( "literalInteger    : %@", [bar literalInteger]);
   mulle_printf( "literalDouble     : %@", [bar literalDouble]);
   mulle_printf( "literalCharacter  : %@", [bar literalCharacter]);
   mulle_printf( "literalArray      : %@", [bar literalArray]);
   mulle_printf( "literalDictionary : %@", [bar literalDictionary]);

// NSStringFromProtocol is a 8.0.0 feature
#if MULLE_FOUNDATION_VERSION  >= ((0 << 20) | (15 << 8) | 0)
   mulle_printf( "literalProtocol   : %@", NSStringFromProtocol( [bar literalProtocol]));
#endif
   mulle_printf( "literalSelector   : %@", NSStringFromSelector( [bar literalSelector]));

   return( 0);
}
```


### Expected output


``` objc
description: VfL Bochum 1848
literalInteger: 1848
literalDouble: 18.480000
literalCharacter: 65
literalArray: (
    1848,
    VfL Bochum
)
literalDictionary: {
    VfL Bochum = 1848;
}
literalSelector: <invalid selector>
```


## Next

Lets examine a how to write a [basic class](mydoc_basics.html) with everything
in it.
