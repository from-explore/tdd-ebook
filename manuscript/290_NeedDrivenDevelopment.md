TODO remember to describe how a TODO list changes!!!

Composition root:

```csharp
 public class Main {
 
     public static void main(String[] args) {
-
+        new TicketOffice();
     }
 
 }
```


Introducing toDto method. But how to pass the Ticket?
TicketOfficeSpecification.java:
```csharp
The Statement shows that we need a factory, so let's introduce it in the Statement first. Name: TicketFactory collaborator.

TicketOfficeSpecification.java

public interface TicketFactory
Time for assertions - what do we expect? Johnny uses his experience to pull a command - because we want to get out of CQS violation. Typically we can do two things - either use a command or a facade. For now mistakenly assuming that the command will take a parameter.
TicketOfficeSpecification.java
```csharp
Also, `Ticket` is an example of collecting parameter pattern. Probably `TicketInProgress` would be a better name.
We know that we need a command, so let's introduce it in the Statement in the GIVEN section.

TicketOfficeSpecification.java
M var ticket = Substitute.For<TicketInProgress>();
Command.java

+public interface Command
+{
+}
Also, this method goes inside the `Command` (already used in the Statement):
+    void Execute(TicketInProgress ticket);
`TicketOffice` needs to know about the command - how will it get it? I decide a command factory will wrap dto with a command (GIVEN section).

TicketOfficeSpecification.java
Introducing the factory declaration.

```csharp
public class TicketOfficeSpecification {
+        var commandFactory = Substitute.For<CommandFactory>();
         commandFactory.CreateBookCommand(reservation)
             .Returns(bookCommand);
```

it means we need an interface:
CommandFactory.java
TicketOfficeSpecification.java

Something's wrong here. Some of these lines were already added:
+   var commandFactory = Substitute.For<CommandFactory>();
    //...
-   TicketInProgress ticket = Substitute.For<TicketInProgress>();
+   var ticket = Substitute.For<TicketInProgress>();

+  public TicketOffice(CommandFactory commandFactory)
+    throw new NotImplementedException("TODO");
Composition root:
+++ b/Java/src/main/java/bootstrap/Main.java
 public class Main {
     public static void main(String[] args) {
-        new TicketOffice();
+        new TicketOffice(new BookingCommandFactory() /* new class - include body or null */);
     }
 }
```
I noticed I can pass the ticket to factory and have the command as something more generic with a void Execute() method. I remove this via refactoring move:
and it disappears from the test as well:

+++ b/Java/src/test/java/TicketOfficeSpecification.java

```csharp
- commandFactory.CreateBookCommand(reservation)
+ commandFactory.CreateBookCommand(reservation, ticket)
      .Returns(bookCommand);

  //...

  //THEN
- bookCommand.Received(1).Execute(ticket);
+ bookCommand.Received(1).Execute();
```

And now I am adding it to the `CreateBookCommand()` method of the factory:
diff --git a/Java/src/main/java/logic/CommandFactory.java 
 public interface CommandFactory
+  Command CreateBookCommand(
+    ReservationRequestDto reservation,
+    TicketInProgress ticket);
TicketOffice should know ticket factory. Adding it to the test:
var ticketOffice = new TicketOffice(
   commandFactory,
and through a quick fix - to the production code.

```csharp
 public class TicketOffice
+  private TicketFactory ticketFactory;

-  public TicketOffice(CommandFactory commandFactory)
+  public TicketOffice(
+        CommandFactory commandFactory,
+        TicketFactory ticketFactory)
-     //todo implement
+    this.ticketFactory = ticketFactory;
+++ b/Java/src/main/java/bootstrap/Main.java

```csharp
 public class Main {

     public static void main(String[] args) {
-        new TicketOffice();
+        new TicketOffice(new BookingCommandFactory(),
+            new TrainTicketFactory()); //TODO new class, or null
     }
 
 }
```
Returning whatever to make sure we fail for the right reason:
  public TicketDto MakeReservation(ReservationRequestDto request)
    //...
    public TicketDto MakeReservation(ReservationRequestDto request)
    {
+       var ticket = ticketFactory.CreateBlankTicket();
+       var command = commandFactory.CreateBookCommand(request, ticket);
+       command.Execute();
        return new TicketDto(null, null, null);
    }
We can remove TODOs? Btw, what about TODO list?
Passed the first assertion but the second one still fails (look at the error message, Luke).
Now we need to make the second assertion pass:
         var ticket = ticketFactory.CreateBlankTicket();
Passed the second assertion, test green. What about the order of invocations?
the empty implementations adds to TODO list.  I need to pick one item to work on. I choose to implement booking command????
+    public TicketInProgress CreateBlankTicket()
    I pick command factory as there is not much I can do with tickets
writing a failing test for a type and dependencies:
This demands new implementation:
+public class BookTicketCommand {
+}
Returning book ticket command forced interface implementation (can be applied via a quick fix)
+++ b/Java/src/main/java/logic/BookingCommandFactory.java
  public Command CreateBookCommand(
    ReservationRequestDto reservation, TicketInProgress ticket)
  {
-   //todo implement
-   return null;
+   return new BookTicketCommand();
  }
The above does not compile yet as the `BookTicketCommand` does not implement a `Command` interface. Need to add it (can be done via quick fix):
-public class BookTicketCommand
+public class BookTicketCommand : Command
+{
+    public void Execute()
+    {
+        //todo implement
 }
Made 2nd assertion pass by introducing field for dto. The 3rd assertion still fails:

    public Command CreateBookCommand(
       ReservationRequestDto reservation,
        TicketInProgress ticket) 
    {
-       return new BookTicketCommand();
+       return new BookTicketCommand(reservation, ticket);
    }
Generating the constructor:
+++ b/Java/src/main/java/logic/BookTicketCommand.java
+    private ReservationRequestDto reservation;
+        this.reservation = reservation;
+    }
(Question: why could we do so many steps? Should we not generate the constructor first and assign the field only when the test fails for the right reason? Probably no, since we already saw it failing for the right reason...)
The test now passes. New items on TODO list (e.g. Execute() method)
(btw, mention here that I tried to specify BookCommand and failed)
/////////////////////////////////???
Discovered TrainRepository interface, because the command will need to get the train from somewhere. Adding it to the test:
+++ b/Java/src/test/java/logic/BookingCommandFactorySpecification.java
@@ -12,10 +12,12 @@ public class BookingCommandFactorySpecification {
     public void ShouldCreateBookTicketCommand() {
-        var bookingCommandFactory = new BookingCommandFactory();
+        var trainRepo = Substitute.For<TrainRepository>();
+        var bookingCommandFactory = new BookingCommandFactory(
+            trainRepo
+        );
         var reservation = Substitute.For<ReservationRequestDto>();
         var ticket = Substitute.For<Ticket>();
-
         //WHEN
         Command result = bookingCommandFactory
             .CreateBookCommand(reservation, ticket);
And the class constructor:
+  public BookingCommandFactory(TrainRepository trainRepo) {
+      //todo implement
+  }

   public Command CreateBookCommand(ReservationRequestDto reservation, Ticket ticket) {
       return new BookTicketCommand(reservation, ticket);
Thus i have discovered a train repository:

(btw, shouldn't I have started with the command? I already have the factory and the command concrete type...)
Discovered train collaborator and getBy repo method. Now what type should the train variable be?
public class BookingCommandFactorySpecification 
{
         var ticket = Substitute.For<TicketInProgress>();

@@ -26,5 +31,7 @@
//////////////TODOOOOOOOOOOOOOOOOO
Discovered a Train interface
First in test:
And introduced using the IDE:
+++ b/Java/src/main/java/logic/Train.java

```csharp
+public interface Train
+{
+}
```

Discovered `GetTrainBy`:

+++ b/Java/src/main/java/logic/TrainRepository.java

```csharp
 public interface TrainRepository {
+    Train GetTrainBy(String trainId);
 }
```
Discovered CouchDbTrainRepository. The TODO list grows
+        ),  new TrainTicketFactory());

We won't be implementing it here... Just call the constructor later.

+public class CouchDbTrainRepository : TrainRepository
+    public Train GetTrainBy(String trainId)
    Made the last assertion from the factory test pass

Adding the dependencies to the factory:

+++ b/Java/src/main/java/logic/BookingCommandFactory.java
 public class BookingCommandFactory : CommandFactory {
-    public BookingCommandFactory(TrainRepository trainRepo) {
-        //todo implement
+    private TrainRepository trainRepo;
 
+    public BookingCommandFactory(TrainRepository trainRepo) {
+        this.trainRepo = trainRepo;
     }
 
     
     public Command CreateBookCommand(ReservationRequestDto reservation, Ticket ticket) {
-        return new BookTicketCommand(reservation, ticket);
+        return new BookTicketCommand(
+            reservation,
+            ticket, trainRepo.GetTrainBy(reservation.trainId));
 }
and the command created with that factory:

```

As my next step, I choose BookTicketCommand

I prefer it over TicketFactory as it will allow me to learn more about the TicketInProgress interface. So now I am optimizing for learning.

+++ b/Java/src/test/java/logic/BookTicketCommandSpecification.java
+public class BookTicketCommandSpecification
+{
+
+}
I have yet to discover what behavior I will require from the command
+public class BookTicketCommandSpecification {
+    [Fact]
+    public void ShouldXXXXXXXXXXXXX()
+    {
+        //GIVEN
+        //WHEN
+        //THEN
+        assertThat(1).isEqualTo(2);
+}
Starting command test
brain dump - just invoke the only existing method
@@ -9,8 +10,10 @@ public class BookTicketCommandSpecification 
{
     public void ShouldXXXXXXXXXXXXX() 
     {
Introducing collaborators and stating expectations
 public class BookTicketCommandSpecification 
 {
     public void ShouldXXXXXXXXXXXXX() 
     {
+        train.Received(1).Reserve(reservation.seatCount, ticket);
The test used a non-existing `Recerve` method - time to introduce it now.
+    void Reserve(int seatCount, TicketInProgress ticketToFill);
Implementation to pass the test:
Made dummy implementation of TrainWithCoaches. We're not test-driving this - Benjamin will go for coffee.

TrainWithCoaches implements an interface, so it has to have the signatures. These empty methods make it to the TODO list.

+
+    public void Reserve(int seatCount, TicketInProgress ticketToFill)
 Renaming a test (should've done this earlier). Should have left a TODO.
As we discovered a new class, time to test-drive it:
+public class TrainWithCoachesSpecification 
+{
This doesn't pass the compilation yet. Time to fill the blanks.
+        var ticket = Substitute.For<TicketInProgress>();

Passed the compiler. Now time for some deeper thinking on the expectation
+++ b/Java/src/test/java/logic/TrainWithCoachesSpecification.java
@@ -13,13 +14,16 @@ public class TrainWithCoachesSpecification
{

         var ticket = Substitute.For<TicketInProgress>();


```
Verifying coaches although none were added yet. Discovered the coach interface:
```csharp
+public interface Coach
+{
+}
```
Time to introduce the coaches. 3 is many:

```csharp
+        var coach1 = Substitute.For<Coach>();
+        var coach2 = Substitute.For<Coach>();
+        var coach3 = Substitute.For<Coach>();
```
Also, discovered the Reserve() method - time to put it in:

```csharp
 public interface Coach 
 {
```
passing coaches as vararg: not test-driving the vararg, using the Kent Beck's putting the right implementation.

```csharp
 public class TrainWithCoaches : Train
+{
+    public TrainWithCoaches(params Coach[] coaches)
+    {
```

This should still pass. now passing the coaches as parameters:



```csharp
@@ -12,12 +12,14 @@ public class TrainWithCoachesSpecification 
{
     public void ShouldXXXXX() 
     { //todo rename

```
Missing the assumptions about whether the coach allows up front reservation:

```csharp

```
Added an item to TODO list - we'll get back to it later. if no coach allows up front, we take the first one that has the limit.
Discovered AllowsUpFrontReservationOf() method.

Introduced the method.  a too late TODO - CouchDbRepository should supply the coaches:


```csharp 
 public interface Coach
 {
```


```csharp

```
gave a good name to the test.


//????????????????????? what is this?

```csharp
@@ -7,6 +7,5 @@ public class TrainWithCoaches : Train
{
     public void Reserve(int seatCount, TicketInProgress ticketInProgress)
     {
```

Now that the scenario is ready, I can give it a good name:


```csharp
```
Implementing the first behavior. (in the book, play with the if and return to see each assertion fail):

```csharp
 public class TrainWithCoaches : Train 
 {
     public TrainWithCoaches(Coach... coaches)
     {

-    public void Reserve(int seatCount, TicketInProgress ticketInProgress) 
     {
+        foreach (var coach in coaches) {
```
//by the way, this can be nicely converted to Linq: coaches.First(c => c.AllowsUpFrontReservationOf(seatCount)).?Reserve(seatCount, ticketInProgress);
Discovered AllowsReservationOf method:

```csharp 
 public class TrainWithCoachesSpecification
 {

+    ShouldReserveSeatsInFirstCoachThatHasFreeSeatsIfNoneAllowsReservationUpFront() 
+    {
```

+++ b/Java/src/main/java/logic/Coach.java
```csharp
@@ -4,4 +4,6 @@ public interface Coach
{
     void Reserve(int seatCount, TicketInProgress ticket);

     bool AllowsUpFrontReservationOf(int seatCount);
+
+    bool AllowsReservationOf(int seatCount);
 }
```
Bad implementation (break; instead of return;) alows the test to pass! Need to fix the first test:

```csharp
@@ -9,6 +9,12 @@ public class TrainWithCoaches : Train
{

     public void Reserve(int seatCount, TicketInProgress ticketInProgress) 
     {
+        foreach (var coach in coaches) 
+        {
+            if(coach.AllowsReservationOf(seatCount)) 
+            {
         foreach (var coach in coaches) 
         {
             if(coach.AllowsUpFrontReservationOf(seatCount))
             {
```

In the following changes, forced the right implementation. But need to refactor the tests. Next time we change this class, we refactor the code:

First we need to say we allow reservations. This dependency between tests is a sign of a design problem.
//TODOOOOOOOOOOOOOOOOOOOOOOOOOOOOO

+++ b/Java/src/test/java/logic/TrainWithCoachesSpecification.java

```csharp
@@ -28,6 +28,12 @@ public class TrainWithCoachesSpecification 
{
    ...
             .Returns(true);
         coach3.AllowsUpFrontReservationOf(seatCount)
             .Returns(true);
+        coach1.AllowsReservationOf(seatCount)
+            .Returns(true);
+        coach2.AllowsReservationOf(seatCount)
+            .Returns(true);
+        coach3.AllowsReservationOf(seatCount)
+            .Returns(true);
 
         //WHEN
         trainWithCoaches.Reserve(seatCount, ticket);
```

```csharp
@@ -10,17 +10,16 @@ public class TrainWithCoaches : Train 
{
     public void Reserve(int seatCount, TicketInProgress ticketInProgress) 
     {
         foreach (var coach in coaches) 
         {
-            if(coach.AllowsReservationOf(seatCount)) 
-            {
+            if(coach.AllowsUpFrontReservationOf(seatCount)) 
+            {
         foreach (var coach in coaches) 
         {
-            if(coach.AllowsUpFrontReservationOf(seatCount)) 
-            {
+            if(coach.AllowsReservationOf(seatCount)) 
             {
```
Refactored coaches (truth be told, I should refactor prod code):

```csharp    
```
Refactored tests. TODO start from this committ to show refactoring!!

```csharp
```
Addressed todo - created a class CoachWithSeats:

```csharp
+    public void Reserve(int seatCount, TicketInProgress ticket) 
+    {
+    public bool AllowsUpFrontReservationOf(int seatCount) 
+    {
+    public bool AllowsReservationOf(int seatCount) 
+    {
```


```csharp
```
Brain dump

```csharp
+        var reservationAllowed = coachWithSeats.AllowsReservationOf(seatCount);
```
Discovered an interface Seat:

```csharp
```


```csharp
         var reservationAllowed = coachWithSeats.AllowsReservationOf(seatCount);
```
Created enough seats:

```csharp
```
Added a constructor: