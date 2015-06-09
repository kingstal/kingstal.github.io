# Core Data

### NSManagedObjectModel  <----->  NSManagedObject

`Entities`、`Attributes`、`Relationships`

### NSManagedObjectContext

> Create a `UIManagedDocument` and ask for its `managedObjectContext` (a `@property`).!




### NSFetchRequest
`Entity`、`NSSortDescriptors`、`NSPredicate`、

    {% highlight objective-c %}
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@“Photo”];
    request.fetchBatchSize = 20;
    request.fetchLimit = 100;
    request.sortDescriptors = @[sortDescriptor];
    request.predicate = ...;
    {% endhighlight %}
