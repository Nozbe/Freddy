## Parsing JSON in Swift

BNRSwiftJSON is a reusable framework for parsing JSON in Swift.
Its primary goal is faciliate the safe parsing of JSON, while also preserving the ease of use presented by parsing JSON in Objective-C.

## Usage

Here are several examples of parsing JSON.
Many of these examples can also be reviewed in the framework's test target: `BNRSwiftJSONTests`.

# Loading Data

The process begins with loading the JSON data.
Our example uses the following JSON:

```
{
    "success": true,
	"people": [
		{
			"name": "Matt Mathias",
            "age": 32,
            "spouse": true,
		},
		{
			"name": "Drew Mathias",
			"age": 33,
            "spouse": true,
		},
        {
            "name": "Sargeant Pepper",
            "age": 25,
            "spouse": false,
        }
	],
	"jobs": [
		"teacher",
		"programmer",
		"judge"
	],
    "states": {
        "Georgia": [
            30301,
            30302,
            30303
        ],
        "Wisconsin": [
            53000,
            53001,
            53002
        ]
    }
}
```

For convenience, we model the loading of JSON from a file located in the framework's target for tests.

```swift
func createData() -> NSData? {
	let testBundle = NSBundle(forClass: BNRSwiftJSONTests.self)
	let path = testBundle.pathForResource("sample", ofType: "JSON")

	if let p = path, u = NSURL(fileURLWithPath: p) {
		return NSData(contentsOfURL: u)
	}
	        
	return nil
}
```

The function `createData()` creates an optional instance of `NSData` containing the JSON above.
In a real app, this function would be replaced by whatever yields the JSON payload, most likely some call to a web service.

# The Motivation

Typically, we would do something like this in Swift to get the JSON data:

```swift
if let data = createData() {
    var error: NSError?
    let stuff = NSJSONSerialization.JSONObjectWithData(data, options: nil, error: &error) as? [String: AnyObject]
	       
    var persons: [Person] = []
    if let people = stuff?["people"] as? [[String: AnyObject]] {
	for person in people {
	    if let name = person["name"] as? String, age = person["age"] as? Int, spouse = person["spouse"] as? Bool {
	        persons.append(Person(name: name, age: age, spouse: spouse))
            }
        }
    }
}
```

Swift 1.2 added an improvement to Optional Binding that allows for multiple bindings in a single `if-let`.
This feature lowers the summit of the above pyramid of doom, but there is still significant nesting.

Furthermore, the above code is difficult to debug.
If any of the above optional bindings fail for some reason, then the result is `nil` and we do not have any data.
We ideally would like the syntax to be clean, while also being able to check informative errors should any arise.

# Using BNRSwiftJSON

`BNRSwiftJSON` is a framework that provides clean syntax, safe typing, and useful information in parsing JSON.
Consider the above example using `BNRSwiftJSON`.

```swift
let data = createData()
let json = JSONValue.createJSONValueFrom(data!)
let peopleArray = json["people"].array
switch peopleArray {
case .Success(let people):
	for person in people {
		let per = Person.createWithJSONValue(person)
		switch per {
		case .Success(let p):
			someContainer.append(p)
		case .Failure(let error):
			println(error) // do something better with the error
		}
	}
case .Failure(let error):
	println(error) // do something better with the error
}
```

The above example demonstrates the safety and ease-of-use that the comes with using `BNRSwiftJSON`. 
`JSONValue` is an enumeration with cases matching each value of JSON that may be returned by a web service.
The method `createWithJSONValueFrom(_:)` takes an instance of `NSData` and returns an instance of `JSONValueResult`.
This type has two cases: `.Success` that will have an associated value of type `JSONValue`, and `.Failure` with an associated value of type `NSError`.
Thus, it will quite apparent of there is or is not JSON data to parse.
Moreover, if there is an error parsing the JSON, then the `.Failure` case will carry with it the associated error information.

Once you have a `JSONValueResult`, you can use various subscriptors and computed properties to get the data.
The code in the above example uses the key `people` to extract the array of persons returned by the web service.
The `array` computed property on `JSONValueResult` returns an instance of the `Result` type, which is similar to `JSONValueResult`.
`Result` has two cases, one for a generic value in the `.Success` case, and another `.Failure` case for error information.
If there is data available for `array` to pull out of the `JSONValueResult` instance, then it will return `Result<[JSONValue]>`.
This return type can be interpreted as a `Result` with potentially an array of `JSONValue`s inside of its `.Success` case.

Next, `peopleArray` is `switch`ed over to determine if there is data.
In the `.Success` case, you can grab the array of `JSONValue`s: `case .Success(let people):`.
This line of code places the array of `JSONValue`s in a constant called `people`.
You can then loop through these `JSONValue`s create instances of some model.

The example above uses a `Person` struct.
Here is its implementation:

```swift
public struct Person: JSONValueDecodable, Printable {
	public let name: String
	public let age: Int
	public let spouse: Bool

	public init(name: String, age: Int, spouse: Bool) {
		self.name = name
		self.age = age
		self.spouse = spouse
	}

	public static func createWithJSONValue(value: JSONValue) -> Result<Person> {
		let name = value["name"].string
		let age = value["age"].int
		let isMarried = value["spouse"].bool

		return name.bind { n in 
			age.bind { a in
				isMarried.map { im in 
					return self.init(name: n, age: a, spouse: im)
				}
			}
		}
	}

	public var description: String {
		return "Name: \(name), age: \(age), married: \(spouse)"
	}
}
```

Notice that the `Person` struct conforms to a protocol: `JSONValueDecodable`.
The protocol requires an implementation of a `static` method to construct an instance of the model: `createWithJSONValue(_:)`.
This method takes an instance of `JSONValue`, which is used to extract the model's relevant data.

Finally, notice that `createWithJSONValue(_:)` returns a `Result` with a `Person` in its `.Success` case's associated value if everything goes well.
The benefit here is that you will know if `createWithJSONValue(_:)` succeeds in making an instance.
If the method does not succeed, then you will know why by checking the associated value in the `.Failure` case.

# A More Elegant Way

The above example is very safe, but is perhaps a little mechanical.
The `for` loop and nested `switch` statement in the first `.Success` case is perahps not ideal.
There is a more elegant way to accomplish the same task.

```swift

```

## Underlying Machinery
