# Intermediate Objective C

<!-- vscode-markdown-toc -->
* [Convert JSON to NSDictionary](#ConvertJSONtoNSDictionary)
* [Convert JSON data into NSData](#ConvertJSONdataintoNSData)
	* [From local JSON file](#FromlocalJSONfile)
	* [From URL](#FromURL)
* [Fetching from RESTful API](#FetchingfromRESTfulAPI)
	* [Get request](#Getrequest)
	* [Post request](#Postrequest)
* [Block syntax in objective-c](#Blocksyntaxinobjective-c)
	* [Syntax](#Syntax)
	* [No return, no parameter](#Noreturnnoparameter)
	* [With return, no parameter](#Withreturnnoparameter)
	* [With arguments](#Witharguments)
	* [block as C function parameter](#blockasCfunctionparameter)
	* [OOP - Block as return value](#OOP-Blockasreturnvalue)
	* [OOP - Block as argument](#OOP-Blockasargument)
* [Concurrency](#Concurrency)
	* [Make task asynchronous using GCD - `dispatch`](#MaketaskasynchronoususingGCD-dispatch)
		* [Dispatch asynchronously on another thread](#Dispatchasynchronouslyonanotherthread)
		* [Dispatch asynchronously on main thread](#Dispatchasynchronouslyonmainthread)
	* [Make task asynchronous using `NSOperation`](#MaketaskasynchronoususingNSOperation)
		* [Add operation using block](#Addoperationusingblock)
		* [Add operation using NSInvocation](#AddoperationusingNSInvocation)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->


## <a name='ConvertJSONtoNSDictionary'></a>Convert JSON to NSDictionary

```objc

/* retrieve local JSON file called example.json */
NSString *filePath = [[NSBundle mainBundle] pathForResource:@"example" ofType:@"json"];

/* load the file into an NSData object called JSONData */
NSError *error = nil;
NSData *JSONData = [NSData dataWithContentsOfFile:filePath options:NSDataReadingMappedIfSafe error:&error];

/* convert data into NSDictionary (the JSON may contain an array or a dictionary) */
id JSONObject = [NSJSONSerialization JSONObjectWithData:JSONData options:NSJSONReadingAllowFragments error:&error];

/* JSON files can contain either array or dictionary */
/*- if array */
NSArray *myJSON = (NSArray *)JSONObject;

/*- if dictionary */
NSDictionary *myJSON = (NSDictionary *)JSONObject;


```

## <a name='ConvertJSONdataintoNSData'></a>Convert JSON data into NSData

### <a name='FromlocalJSONfile'></a>From local JSON file

```objc

/* retrieve local JSON file called example.json */
NSString *filePath = [[NSBundle mainBundle] pathForResource:@"example" ofType:@"json"];

/* load the file into an NSData object called JSONData */
NSError *error = nil;
NSData *JSONData = [NSData dataWithContentsOfFile:filePath options:NSDataReadingMappedIfSafe error:&error];

```

### <a name='FromURL'></a>From URL

```objc

/* make a URL object */
NSURL *internetPath = [NSURL URLWithString:@"http://data.dev8d.org/2012/programme/?event=CS41&output=json"];

/* load JSON from the web into an NSData object called JSONData */
NSError *error = nil;
NSData *JSONData = [NSData dataWithContentsOfURL:internetPath options:NSDataReadingMappedIfSafe error:&error];

```

## <a name='FetchingfromRESTfulAPI'></a>Fetching from RESTful API


### <a name='Getrequest'></a>Get request

```objc

NSString *requestString = [NSString stringWithString:@"http://example.com/api/mydata"];
NSMutableURLRequest *urlRequest = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:requestString]];

/* set method to "GET" */
[urlRequest setHTTPMethod:@"GET"];

/* create request session */
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:urlRequest completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    /* if response status is a success */
    NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
    if(httpResponse.statusCode == 200) {
        NSError *parseError = nil;
        NSDictionary *responseDictionary = [NSJSONSerialization JSONObjectWithData:data options:0 error:&parseError];
        NSLog(@"The response is - %@", responseDictionary);
    } else {
        NSLog(@"Error");
    }
}];

/* run */
[dataTask resume];

```

### <a name='Postrequest'></a>Post request

```objc

NSDictionary *body = @{
    @"member_account": @"Linley",
    @"bank_account_no": @"123892",
    @"risk_name": @"popopo",
    @"page_no": @"1",
    @"contact_number": @"1234567890"
};

NSError *error;
NSData *jsonData = [NSJSONSerialization dataWithJSONObject:body options:0 error:&error];

NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];

/* set request URL */
NSString *requestString = @"http://api.testing.com/api/testendpoint";
[request setURL:[NSURL URLWithString:requestString]];

/* set request METHOD */
[request setHTTPMethod:@"POST"];

/* set request BODY */
[request setHTTPBody:jsonData];

/* set request HEADERS */
[request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
[request setValue:@"application/json" forHTTPHeaderField:@"Accept"];
[request setValue:@"" forHTTPHeaderField:@"Authorization"];

/* configure request session */
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)] {

    if (error) {
        NSLog(@"error: %@", error);
    }
    if (!data) {
        return;
    }

    NSError *parseError;

    NSDictionary *responseDataDict = [NSJSONSerialization JSONObjectWithData:data option:0 error:&parseError];

    if (responseDataDict) {
        NSLog(@"requestReply: %@", responseDataDict);
    } else {
        NSLog(@"parseError: %@", parseError);
        NSLog(@"response: %@", response);
        NSString *requestReply = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"requestReply: %@", requestReply);
    }
    
/* start request session */
[dataTask resume];

}


```

## <a name='Blocksyntaxinobjective-c'></a>Block syntax in objective-c

### <a name='Syntax'></a>Syntax

```
ReturnType (^blockName)(parameterTypes) = ^ ReturnType(parameterTypes and parameterNames) {};

```

### <a name='Noreturnnoparameter'></a>No return, no parameter

```objc

void(^myBlock)(void) = ^ {
    NSLog(@"Hello block");
};

myBlock();

```

### <a name='Withreturnnoparameter'></a>With return, no parameter

```objc

int(^myBlock)(void) = ^ int(void) {
    int x = 1;
    int y = 2;
}

NSLog(@"myBlock value: %i", myBlock());

```

### <a name='Witharguments'></a>With arguments

```objc

void(^myBlock)(int, int) = ^ void(int x, int y) {
    NSLog(@"Sum = %i, x + y);
}

myBlock(1, 2);

```

### <a name='blockasCfunctionparameter'></a>block as C function parameter

```objc

/* block without return and without parameters */
void myFunc(void (^myBlock)(void)) {
    NSString *name = myBlock();
    NSLog(@"my name is %@", name);
}

/* block with return and without parameters */
void myFunc(NSString * (^myBlock)(void)) {
    NSString *name = myBlock();
    NSLog(@"my name is %@", name);
}

/* block without return, but with parameters */
void myFunc(void (^myBlock)(NSString *name, NSInteger age)) {
    myBlock(@"deks", 27);
}

```

Example 1 - via substitution

```objc

#import <Foundation/Foundation.h>
void myFunc(void (^myBlock)(NSString *name, NSInteger age)) {
    myBlock(@"deks", 27);
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        void (^myCustomBlock)(NSString *, NSInteger) = ^ void (NSString *name, NSInteger age) {
            NSLog(@"Name: %@\nAge: %lu", name, age);
        };
        myFunc(myCustomBlock);
    }
    return 0;
}
```

Example 2 - direct

```objc

#import <Foundation/Foundation.h>

void myFunc(void (^myBlock)(NSString *name, NSInteger age)) {
    myBlock(@"deks", 27);
}

int main (int argc, const char *argv[]) {
    @autoreleasepool {
        myFunc(^(NSString *name, NSInteger age) {
            NSLog(@"Name: %@\nAge: %lu", name, age);
        });
    }
    return 0;
}

```

### <a name='OOP-Blockasreturnvalue'></a>OOP - Block as return value

```objc
/* no return, no parameter block */
- (void (^) (void))sayHello;

/* no return, with single parameter block */
- (void (^) (BOOL))sayHello;

/* with return, no parameter block */
- (NSString * (^) (void))returnHello;

/* with return, with single parameter block  */
- (NSString * (^) (BOOL))returnHello;

/*- Example */
- (NSString * (^) (BOOL))returnHello {
    return ^ NSString * (BOOL shouldReturnHello) {
        if (shouldReturnHello) return @"hello";
        return @"bar";
    }
}

```

### <a name='OOP-Blockasargument'></a>OOP - Block as argument

```objc

- (void)sayHelloWithBlock:(void (^) (void))block;

- (void)sayHelloWithBlock:(void (^) (BOOL))block;

- (void)sayHelloWithBlock:(NSString * (^) (void))block;

- (void)sayHelloWithBlock:(NSString * (^) (BOOL))block;


```

## <a name='Concurrency'></a>Concurrency

### <a name='MaketaskasynchronoususingGCD-dispatch'></a>Make task asynchronous using GCD - `dispatch`

#### <a name='Dispatchasynchronouslyonanotherthread'></a>Dispatch asynchronously on another thread

```objc

/* get global queue so it will be dispatched on that thread and store in a variable */
dispatch_queue_t *queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

/* dispatch asynchronously the code inside */
dispatch_async(queue, ^ {
    /* background thread */
    /* insert code here */
})

```

#### <a name='Dispatchasynchronouslyonmainthread'></a>Dispatch asynchronously on main thread

```objc

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void) {
    /* background thread */

    dispatch_async(dispatch_get_main_queue(), ^(void) {
        /* run on main thread */
    });
});

```

### <a name='MaketaskasynchronoususingNSOperation'></a>Make task asynchronous using `NSOperation`

#### <a name='Addoperationusingblock'></a>Add operation using block

```objc

NSOperationQueue *q = [[NSOperationQueue alloc] init];

[q addOperationWithBlock: ^{
    /* code to execute in background */
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        /* code to execute in mainqueue */
    }];
}];

```

#### <a name='AddoperationusingNSInvocation'></a>Add operation using NSInvocation

```objc
/* Use NSInvocationOperation when you have a method ready */

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    NSInvocationOperation *displayNumbers = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(displayNumbers) object:nil];

    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    [queue addOperation:displayNumbers];
}

- (void)displayNumbers {
    for (int i = 0; i < 10; i++) {
        NSLog(@"%i", i);
    }
}

@end


```
