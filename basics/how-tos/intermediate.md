# Intermediate Objective C

<!-- vscode-markdown-toc -->
* [Convert JSON to NSDictionary](#ConvertJSONtoNSDictionary)
* [Convert JSON data into NSData](#ConvertJSONdataintoNSData)
	* [From local JSON file](#FromlocalJSONfile)
	* [From URL](#FromURL)
	* [From URL](#FromURL-1)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->


## <a name='ConvertJSONtoNSDictionary'></a>Convert JSON to NSDictionary

```objc

/* retrieve local JSON file called example.json */
NSString * filePath = [[NSBundle mainBundle] pathForResource:@"example" ofType:@"json"];

/* load the file into an NSData object called JSONData */
NSError * error = nil;
NSData * JSONData = [NSData dataWithContentsOfFile:filePath options:NSDataReadingMappedIfSafe error:&error];

/* convert data into NSDictionary (the JSON may contain an array or a dictionary) */
id JSONObject = [NSJSONSerialization JSONObjectWithData:JSONData options:NSJSONReadingAllowFragments error:&error];

/* JSON files can contain either array or dictionary */
/*- if array */
NSArray * myJSON = (NSArray *)JSONObject;

/*- if dictionary */
NSDictionary * myJSON = (NSDictionary *)JSONObject;


```

## <a name='ConvertJSONdataintoNSData'></a>Convert JSON data into NSData

### <a name='FromlocalJSONfile'></a>From local JSON file

```objc

/* retrieve local JSON file called example.json */
NSString * filePath = [[NSBundle mainBundle] pathForResource:@"example" ofType:@"json"];

/* load the file into an NSData object called JSONData */
NSError * error = nil;
NSData * JSONData = [NSData dataWithContentsOfFile:filePath options:NSDataReadingMappedIfSafe error:&error];

```

### <a name='FromURL'></a>From URL

```objc

/* make a URL object */
NSURL * internetPath = [NSURL URLWithString:@"http://data.dev8d.org/2012/programme/?event=CS41&output=json"];

/* load JSON from the web into an NSData object called JSONData */
NSError * error = nil;
NSData * JSONData = [NSData dataWithContentsOfURL:internetPath options:NSDataReadingMappedIfSafe error:&error];

```

## Fetching from RESTful API

```objc

NSString * requestString = [NSString stringWithString:@"http://example.com/api/mydata"];
NSMutableURLRequest *urlRequest = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:requestString]];

/* set method to "GET" */
[urlRequest setHTTPMethod:@"GET"];

/* create request session */
NSURLSession * session = [NSURLSession sharedSession];
NSURLSessionDataTask * dataTask = [session dataTaskWithRequest:urlRequest completionHandler:^(NSData * data, NSURLResponse * response, NSError * error) {
    /* if response status is a success */
    NSHTTPURLResponse * httpResponse = (NSHTTPURLResponse *)response;
    if(httpResponse.statusCode == 200) {
        NSError * parseError = nil;
        NSDictionary * responseDictionary = [NSJSONSerialization JSONObjectWithData:data options:0 error:&parseError];
        NSLog(@"The response is - %@", responseDictionary);
    } else {
        NSLog(@"Error");
    }
}];

/* run */
[dataTask resume];

```
