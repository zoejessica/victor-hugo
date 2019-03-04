---
date: "2019-03-04"
draft: false
title: "Snapshotting approximate values"
slug: snapshotting-approximate-values

categories:
- Testing
tags:
- xcode
- swift
- testing
- snapshot
description: Using a snapshot library to perform comparison between floating point values
---
In my former stockbroking life, sometimes you'd be looking at the bids and offers for a stock, when something weird would happen. All of a sudden, the offers to sell the stock at various prices would just disappear. In one instantaneous trade, someone out there had bought an incredible number of shares *at market*, i.e. immediately, taking any and all offers without regard to price. The stock price would spike up a lot, but there’d be no chatter or news to explain why someone might be buying so much, so fast.

There’d be a second of confusion before we understood: someone must have had a sausage-fingered moment or, ahem, put their elbow on the keyboard, executed a trade, and was about to have a pretty bad day.

It used to be quite easy to do. 🤦🏼‍♀️

Luckily, now I'm a developer there are tests to protect me from myself...

### So much gelato, so many numbers

I’m writing an app that mathematically generates recipes for ice-cream.
I wanted some way to iterate through a set of recipe calculations and get alerted if I've messed up.
There are a lot of steps that I *could*  sausage-finger, and so write laborious tests for
— validating the json data, decoding json, constraints I set up on the calculation, the results of the calculation.
But here I just wanted a quick-and-dirty, macro check.

My first thought was to dump the calculation output to text and use the [Point Free](https://www.pointfree.co) snapshotting library to diff the result.
(The video series on this is great, [the overview is free to watch](https://www.pointfree.co/episodes/ep41-a-tour-of-snapshot-testing), and [the code is all open source](https://github.com/pointfreeco/swift-snapshot-testing)).

Trouble is, those calculations produce floating point values, and I don’t have control of the random seed that’s being used.
So the values are a tiny bit different every time, which means so is the text, and the snapshot fails on those tiny discrepancies.  

Still playing with this library, I wrote a custom snapshotting strategy that takes input in the form of a `[String : Double]` dictionary, as well as a percentage tolerance value.

``` swift
public func approximateValues(tolerance percentage: Double) -> SimplySnapshotting<NamesToValues> {
    let diffingStrategy = Diffing.approximateValues(tolerance: percentage)
    return SimplySnapshotting.init(pathExtension: “json”, diffing: diffingStrategy)
}
```

The reference value is recorded to disk as a json object.
When the test is run, the old values can be properly reconstituted for comparison.

When the diff is called with old recorded and new test values, it firsts checks if all the keys match. If not, it calls out to the `Diffing.lines.diff` string diffing function that’s part of the library. This produces a standard error message, which is ignored, but also a nice readable diff of the old and new keys in the form of an `XCTAttachment`:

```swift
// If the dictionary keys don't match,
// just return the string diff of the keys
guard oldDict.keys == newDict.keys else {
    let (_ , attachment) = linesDiffer(oldDict.keys.joined(separator: "\n"),
		newDict.keys.joined(separator: "\n"))!
    return ("Keys did not match", attachment)
}
```

Then it compares the float values for each key to see if they’re within the given accuracy tolerance. If they are too far apart, it produces a custom error message and uses the `lines` differ again to produce a readable attachment. This time it uses all the recorded and test data to produce a string diff, passing through JSON encoding to strings, to get a consistent format:

```swift
private static func diff(_ old: [String : Double], _ new: [String : Double]) -> (String, [XCTAttachment])? {
		let oldString = try! String(decoding: encoder.encode(old), as: UTF8.self)
		let newString =  try! String(decoding: encoder.encode(new), as: UTF8.self)
		return linesDiffer(oldString, newString)
}
```

I’m using `pullback` to make the strategy more ergonomic to use with my data models, and setting tolerances according to the type of data at the same time:

```swift
let composition: Snapshotting<FancyDataType, [String : Double]> = approximateValues(tolerance: 2.0).pullback { fancyData in
    // ... transformation here ...
		return simpleData // [String : Double]
}
```

Naming the snapshot at the call site is a nice touch when iterating through many similar calculations, as it shows up inline as part of the test failure message:

```swift
assertSnapshot(matching: solution.composition, as: composition, named: variety.name)
```

Of course there are other ways that I could have built this out, using `XCTAssertEqual(_:_:accuracy)`, for example. But I like having a simple diff on names and values as a top level warning system, and it’s easy to adapt to changing data models.

[Here’s the full gist.](https://gist.github.com/zoejessica/074d4bf7f378afbc1979e3316539eaff)
