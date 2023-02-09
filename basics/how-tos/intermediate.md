# Intermediate Objective C

- [Intermediate Objective C](#intermediate-objective-c)
  - [Convert JSON to NSDictionary](#convert-json-to-nsdictionary)
  - [Convert JSON data into NSData](#convert-json-data-into-nsdata)
    - [From local JSON file](#from-local-json-file)
    - [From URL](#from-url)
  - [Fetching from RESTful API](#fetching-from-restful-api)
    - [Get request](#get-request)
    - [Post request](#post-request)
  - [Block syntax in objective-c](#block-syntax-in-objective-c)
    - [Syntax](#syntax)
    - [No return, no parameter](#no-return-no-parameter)
    - [With return, no parameter](#with-return-no-parameter)
    - [With arguments](#with-arguments)
    - [block as C function parameter](#block-as-c-function-parameter)
    - [OOP - Block as return value](#oop---block-as-return-value)
    - [OOP - Block as argument](#oop---block-as-argument)
  - [Concurrency](#concurrency)
    - [Make task asynchronous using GCD - `dispatch`](#make-task-asynchronous-using-gcd---dispatch)
      - [Dispatch asynchronously on another thread](#dispatch-asynchronously-on-another-thread)
      - [Dispatch asynchronously on main thread](#dispatch-asynchronously-on-main-thread)
    - [Make task asynchronous using `NSOperation`](#make-task-asynchronous-using-nsoperation)
      - [Add operation using block](#add-operation-using-block)
      - [Add operation using NSInvocation](#add-operation-using-nsinvocation)
    - [Add operation using NSBlockOperation](#add-operation-using-nsblockoperation)
    - [Add operation using NSOperation](#add-operation-using-nsoperation)
  - [NSPredicate](#nspredicate)
    - [Check if string conforms with the pattern](#check-if-string-conforms-with-the-pattern)
    - [Filter array with predicate method 1](#filter-array-with-predicate-method-1)
    - [Filter array with predicate method 2](#filter-array-with-predicate-method-2)
  - [NSRegularExpression](#nsregularexpression)
    - [Check if string is valid](#check-if-string-is-valid)
    - [See matched substring with pattern](#see-matched-substring-with-pattern)
  - [How to make your own PLIST](#how-to-make-your-own-plist)
    - [Accessing PLIST key and values](#accessing-plist-key-and-values)
    - [Accessing plist with validation if file exists](#accessing-plist-with-validation-if-file-exists)
    - [Accessing plist with singleton pattern](#accessing-plist-with-singleton-pattern)
  - [How to get components of UIColor](#how-to-get-components-of-uicolor)

## Convert JSON to NSDictionary

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

## Convert JSON data into NSData

### From local JSON file

```objc

/* retrieve local JSON file called example.json */
NSString *filePath = [[NSBundle mainBundle] pathForResource:@"example" ofType:@"json"];

/* load the file into an NSData object called JSONData */
NSError *error = nil;
NSData *JSONData = [NSData dataWithContentsOfFile:filePath options:NSDataReadingMappedIfSafe error:&error];

```

### From URL

```objc

/* make a URL object */
NSURL *internetPath = [NSURL URLWithString:@"http://data.dev8d.org/2012/programme/?event=CS41&output=json"];

/* load JSON from the web into an NSData object called JSONData */
NSError *error = nil;
NSData *JSONData = [NSData dataWithContentsOfURL:internetPath options:NSDataReadingMappedIfSafe error:&error];

```

## Fetching from RESTful API

### Get request

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

### Post request

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

## Block syntax in objective-c

### Syntax

```objc
ReturnType (^blockName)(parameterTypes) = ^ ReturnType(parameterTypes and parameterNames) {};

```

### No return, no parameter

```objc

void(^myBlock)(void) = ^ {
    NSLog(@"Hello block");
};

myBlock();

```

### With return, no parameter

```objc

int(^myBlock)(void) = ^ int(void) {
    int x = 1;
    int y = 2;
}

NSLog(@"myBlock value: %i", myBlock());

```

### With arguments

```objc

void(^myBlock)(int, int) = ^ void(int x, int y) {
    NSLog(@"Sum = %i, x + y);
}

myBlock(1, 2);

```

### block as C function parameter

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

### OOP - Block as return value

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

### OOP - Block as argument

```objc

- (void)sayHelloWithBlock:(void (^) (void))block;

- (void)sayHelloWithBlock:(void (^) (BOOL))block;

- (void)sayHelloWithBlock:(NSString * (^) (void))block;

- (void)sayHelloWithBlock:(NSString * (^) (BOOL))block;


```

## Concurrency

### Make task asynchronous using GCD - `dispatch`

#### Dispatch asynchronously on another thread

```objc

/* get global queue so it will be dispatched on that thread and store in a variable */
dispatch_queue_t *queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

/* dispatch asynchronously the code inside */
dispatch_async(queue, ^ {
    /* background thread */
    /* insert code here */
})

```

#### Dispatch asynchronously on main thread

```objc

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void) {
    /* background thread */

    dispatch_async(dispatch_get_main_queue(), ^(void) {
        /* run on main thread */
    });
});

```

### Make task asynchronous using `NSOperation`

#### Add operation using block

```objc

NSOperationQueue *q = [[NSOperationQueue alloc] init];

[q addOperationWithBlock: ^{
    /* code to execute in background */
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        /* code to execute in mainqueue */
    }];
}];

```

#### Add operation using NSInvocation

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

### Add operation using NSBlockOperation

```objc

/* version 1 - simple */
NSBlockOperation * fetchOperation = [NSBlockOperation blockOperationWithBlock:^{
    /* code to execute in background */
}];

/* version 2 - adding more blocks */
NSBlockOperation * fetchOperation = [NSBlockOperation blockOperationWithBlock:^{
    /* code to execute in background */
}];

[fetchOperation addExecutionBlock:^{
    /* add this block to operation */
}];

NSOperationQueue * queue = [[NSOperationQueue alloc] init];
queue.name = @"fetch";
[queue addOperation:fetchOperation];


```

### Add operation using NSOperation

...to be created

## NSPredicate

### Check if string conforms with the pattern

```objc

NSString *string = @"world_cup39@3x.png";
NSString *pattern = @"[a-z]+[_a-z0-9]*@3x\\.png";

NSPredicate * predicate = [NSPredicate predicateWithFormat:@"SELF matches %@", pattern];

BOOL isValid = [predicate evaluateWithObject:string];

NSLog(@"is valid: %d", isValid);

```

### Filter array with predicate method 1

```objc

NSArray * trainees = @[
    @{@"name": @"dekideks", @"team": @"iOS"},
    @{@"name": @"sanji", @"team": @"iOS"},
    @{@"name": @"shinmon", @"team", @"iOS"},
    @{@"name": @"omen", @"team": @"iOS"},
    @{@"name": @"teteng", @"team": @"iOS"},
    @{@"name": @"vanilla", @"team": @"iOS"},
];

NSString * searchString = @"e";

NSPredicate * predicate2 = [NSPredicate predicateWithFormat:@"name contains[c] %@", searchString];

NSLog(@"%@", [trainees filteredArrayUsingPredicate:predicate2]);

```

### Filter array with predicate method 2

```objc

NSArray * trainees = @[
    @{@"name": @"dekideks", @"team": @"iOS"},
    @{@"name": @"sanji", @"team": @"iOS"},
    @{@"name": @"shinmon", @"team": @"iOS"},
    @{@"name": @"omen", @"team": @"iOS"},
    @{@"name": @"teteng", @"team": @"iOS"},
    @{@"name": @"vanilla", @"team": @"iOS"},
];

NSString * searchString = @"e";

NSPredicate * predicate = [NSPredicate predicateWithBlock:^BOOL(id _Nullable evaluatedObject, NSDictionary<NSString *, id> * _Nullable bindings) {

    if ([searchString isEqualToString:[evaluatedObject objectForKey:@"name"]] || [[evaluatedObject objectForKey:@"name"] rangeOfString:searchString].location != NSNotFound) {
        return evaluatedObject;
    }

    return nil;
}];

NSLog(@"%@", [trainees filteredArrayUsingPredicate:predicate]);

```

## NSRegularExpression

### Check if string is valid

```objc

NSString *string = @"48";
NSString *pattern = @"^\\d{2}$";

NSError *error;
NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:pattern options:0 error:&error];

/* check if valid */
NSInteger numberOfMatches = [regex numberOfMatchesInString:string options:0 range:NSMakeRange(0, string.length)];

BOOL isValid = NO;

if (numberOfMatches > 0) isValid = YES;

NSLog(@"is valid: %d", isValid);

```

### See matched substring with pattern

```objc

NSString *string = @"48 59 28";

NSString *pattern = @"\\d{2}";

NSError *error;
NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:pattern options:0 error:&error];
NSArray<NSTextCheckingResult *> *matches = [regex matchesInString:string options:0 range:NSMakeRange(0, string.length)];

NSMutableArray<NSString *> *matchedStrings = [[NSMutableArray alloc] init];

for (NSTextCheckingResult *result in matches) {
    NSString * matchedString = [string substringWithRange:[results range]];

    [matchedStrings addObject:matchedString];
}

NSLog(@"matches: %@", matchedString);

```

## How to make your own PLIST

1. Right-click the folder where you want to add a plist
2. Choose New File
3. Choose iOS > Resource > Property List
4. Set the name you want
5. Go to the newly created file
6. Add key and value as you desired

### Accessing PLIST key and values

```objc

/* sample: plist file name is metadeks.plist */

NSString *filePath = [[NSBundle mainBundle] pathForResource:@"metadeks" ofType:@"plist"];
NSDictionary *contents = [NSDictionary dictionaryWithContentsOfFile:filePath];

NSLog(@"%@", contents);

```

### Accessing plist with validation if file exists

```objc

NSString *filePath = [[NSBundle mainBundle] pathForResource:@"metadeks" ofType:@"plist"];

NSDictionary *contents = nil;

/* check if file exist first */
NSFileManager *fm = [NSFileManager defaultManager];

if ([fm fileExistsAthPath:filePath]) {
    contents = [NSDictionary dictionaryWithContentsOfFile:filePath];
}

NSLog(@"%@", contents);

```

### Accessing plist with singleton pattern

```objc

- (void)viewDidLoad {
    [super viewDidLoad];

    NSDictionary *metadata = [self metadata];
    NSLog(@"%@", metadata);
}


- (NSDictionary *)metadata {
    static NSDictionary * metadata = nil;
    static dispatch_once_t onceToken;

    dispatch_once(&onceToken, ^{
        NSString *filePath = [[NSBundle mainBundle] pathForResource:@"metadeks" ofType:@"plist"];
        NSFileManager *fm = [NSFileManager defaultManager];

        if ([fm fileExistsAtPath:filePath]) {
            metadata = [NSDictionary dictionaryWithContentsOfFile:filePath];
        }
    });
    return metadata;
}

```

## How to get components of UIColor

```objc

UIColor *steelBlueColor = [UIColor colorWithRed:0.3f green:0.4f blue:0.6f alpha:1.0f];

/* get components */

CGColorRef colorRef = steelBlueColor.CGColor;

const CGFloat *components = CGColorGetComponents(colorRef);

NSUinteger componentsCount = CGColorGetNumberOfComponents(colorRef);

for (NSUInteger counter = 0; counter < componentsCount; counter++) {
    NSLog(@"Component %lu = %.02f", counter, components[counter]);
}

```
