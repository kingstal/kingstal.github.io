## Archiving


    {% highlight objective-c %}

    @protocol NSCoding
    - (void)encodeWithCoder:(NSCoder *)aCoder;
    - (instancetype)initWithCoder:(NSCoder *)aDecoder;
    @end

    [NSKeyedArchiver archiveRootObject:self.privateItems toFile:path]

    self.privateItems = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
    {% endhighlight %}


## NSData

    {% highlight objective-c %}

    NSData* data = UIImageJPEGRepresentation(image, 0.5);
    [data writeToFile:imagePath atomically:YES];

    result = [UIImage imageWithContentsOfFile:imagePath];

    [[NSFileManager defaultManager] removeItemAtPath:imagePath error:nil];
    {% endhighlight %}


## The Application Bundle 只读

    {% highlight objective-c %}
    // Get a pointer to the application bundle
    NSBundle *applicationBundle = [NSBundle mainBundle];
    // Ask for the path to a resource named myImage.png in the bundle
    NSString *path = [applicationBundle pathForResource:@"myImage"
                                                 ofType:@"png"];
    {% endhighlight %}
