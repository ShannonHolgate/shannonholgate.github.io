---
layout: post
title: "Objective-C Blocks"
description: "Quick guide on using blocks in Objective-C"
category: articles
tags: [Objective-C]
comments: true
---
## Blocks

In Objective-C a block is an **inline anonymous collection of code** - this means it is an anonymous function.
An anonymous function is a function that isn't nescessarily bound to an identifier. Don't worry if this doesn't click with you yet, I will explain it as we move through the guide.

<pre><code class="language-clike">int (^addition)(int,int) = 
		^(int num1, int num2) {return num1 + num2};</code></pre>

The [Developer API] states that a block has the following properties:

* A typed argument list like a function
* Inferred (x+1 could be int) or declared return type
* Can capture state from the lexical scope where it's defined (Note: *lexical scope* is closure or {} scope)
* Can modify the state of the function (lexical scope again)
* Can share the potential for modification with other blocks in the function
* Can continue to process after the lexical scope has been destroyed

### How to use Blocks

The simplest block you can make is much like a void return function, it simply runs a snippet of code when called:

<pre><code class="language-clike">void (^simple)(void) = 
		^{NSLog(@"You just called a block!");};</code></pre>

This is a **void return** block, it takes no parameters and returns nothing. It simply prints to the console when <code class="language-clike">simple();</code> is called. The first <code class="language-clike">void</code> signifies the block has no return type and the second <code class="language-clike">(void)</code> shows the block has no parameters. If you compare this with a regular function you would think the block could be written as: <code class="language-clike">void (^simple)()</code> by leaving out the second <code class="language-clike">(void)</code>. This is not the case as blocks that take no parameters **must** specify the <code class="language-clike">(void)</code> parameter.<br/><br/>
Now a slighly more complicated block variable:
<pre><code class="language-clike">int (^addition)(int,int) = 
		^(int num1, int num2) {return num1 + num2};</code></pre>

___

Lets break this down into each part:

* <code class="language-clike">int</code>					Return type of the declared block
* <code class="language-clike">(^addition)</code>			The name of the block (much like a function name)
* <code class="language-clike">(int,int)</code>				Parameter list definition(much like a function header)
* <code class="language-clike">=</code>                     Simple assignment operator to give the block an reference name
* <code class="language-clike">^</code>						Caret symbol again to declare the next statement as a block
* <code class="language-clike">(int num1, int num2)</code>	Input parameter list to match the definition
* <code class="language-clike">{return num1 + num2};</code>	Block body, this acts just like a function body

Note: This block *is* still an **anonymous block**, by saying the block has a name we are merely saying the name *references* an **anonymous block**

That’s it! It looks complicated but if you break it down and look at the block like a function you can make sense of it pretty quickly.
Have a look at [Declaring iOS blocks] in the developer library to further explain what I have talked about here.

___

So now that we’ve declared the block above, let's put it to use: 

<pre><code class="language-clike">int (^addition)(int,int) = 
		^(int num1, int num2) {return num1 + num2};

// Call the block above just like a function
int result = addition(5,25);

NSLog(@"result = %i", result); 
//This prints as "result = 30"</code></pre>

Called just like a function the block allows some simple function calls. However, a block is much more powerful than this.
We can use blocks as callbacks from functions. A callback is a set of commands or portion of code we want to run when nescessary in our function (when something fails, for example). 
The callback should change and be manipulated within the function to give us the most informative callback possible.
To do this we want to use a block as a parameter in our function - this way we can specify some anonymous function to run when needed.


Take the following example:

<pre><code class="language-clike">- (void)printNumbers:(int)first andInt:(int)second 
		withBlock:(int(^)(int,int))sub;</code></pre>

looking closely at the final parameter you can see the block header being declared:

* <code class="language-clike">(int(^)(int,int))sub</code>		This is how we define a block as a parameter
* <code class="language-clike">int(^addition)(int,int)</code>	This is how we define a standalone block

The structure is the same except we take the block name and move it to the outside of the block to allow the method to read well when it’s called.

The method body for "printNumbers" could be something like below

<pre><code class="language-clike">- (void)printNumbers:(int)first andInt:(int)second 
		withBlock:(int(^)(int,int))sub
{
	//Nothing fancy here, just a couple of logs
	NSLog(@"first number = %i",first); 
	NSLog(@"second number = %i",second); 

	//Then call the block just like before
	int blockResult = sub(first, second);
	NSLog(@"blockResult = %i",blockResult);
}</code></pre>

So nothing special has happened yet - we are just printing a couple of parameters then calling the block we’ve declared in the header.
So let's move onto implementation so the blocks can really shine.

<pre class="code-box"><code class="language-clike">- (void)viewDidLoad
{
	[self printNumbers:30 andInt:10 
			withBlock:^(int first, int second){
    	// "sub" Block declared inline
    	return first - second;
    }];
}</code></pre>

You can manipulate your variables anyway you want within the block, the beauty being that it’s functionality is called at the precise moment that it’s referenced in the underlying method.

___

To take things a step further we can introduce **__block** variables which can be used inside the block to modify the current lexical scope.
__block variables are just like regular variables except the compiler knows to preserve them if the block is copied to the heap. This allows the variable to be mutable within the block. Regular variables cannot be modified from within the block.

<pre><code class="language-clike">- (void)printNumbers:(int)first andInt:(int)second 
		withBlock:(int(^)(int,int))sub
{
	//Nothing fancy here, just a couple of logs
	NSLog(@"first number = %i",first); 
	NSLog(@"second number = %i",second); 

	//Then call the block just like before
	int blockResult = sub(first, second);
	NSLog(@"blockResult = %i",blockResult);
}

- (void)viewDidLoad
{
	__block int halfOfFirstNumber;
    [self printNumbers:30 andInt:10 
    		withBlock:^(int first, int second){
		// "sub" Block declared inline
		halfOfFirstNumber = first/2;
        return first - second; 
    }];

	NSLog(@"half Number = %i",halfOfFirstNumber); 
	//This prints as "halfOfFirstNumber = 15"
    
}</code></pre>

### Why use Blocks?
Blocks are powerful! They allow us to refer to values even after the current method goes out of scope. Maybe you need to access a variable and store it on exit of a method? Just kick up a block and it will hold the value. Then when you ask for it, you can assign that value to somewhere else even the method holding the value has finished - this can all be happening after the method has ended.


I find them very useful when I want to use a method which has the potential to fail in several ways, such as a user not being logged in or a user that doesn't have admin rights. That's two different ways to fail that I want to handle with a single block.

**Take a look at a real world example below:**

___


#### Helper.h

<pre><code class="language-clike">@interface Helper: NSObject

+ (NSArray *)getAllMessagesForCurrentUser:(void(^)(int))failure;

@end
</code></pre>

#### Helper.m

We will declare a method in Helper.m to get some messages from a currentUser object. The method should throw a simple error through a block if no user is logged in.

<pre><code class="language-clike">#import "Helper.h"
#import "User.h"
#import "Message.h"
@implementation Helper

static User *currentUser;

+ (NSArray *)getAllMessagesForCurrentUser:(void(^)(int))failure
{
    NSMutableArray *messages = [NSMutableArray array];

    if (currentUser == nil)
    {
        NSLog(@"User not logged in");

        /**
        * Call the failure block because there is no current user
        * The quirk here is that the failure block hasn't been
        * defined yet, this is because the block is an anonymous 
        * function and we'll define it when we call the 
        * getAllMessagesForCurrentUser function.
        **/
        failure(1);
        
        return nil;
    }
    else
    {
        /**
        * Here's an example of an iOS method provided by the SDK.
        * This method uses makes use of a callback block which we 
        * define ourselves. In this case I want to get each message
        * and store it in a local array to use later.

        [currentUser.messages 
        		enumerateObjectsUsingBlock:^(id obj, BOOL *stop){
            Message *message = obj;
            [messages addObject:message];
        }];
        
        return messages;
    }
}

+ (User *)currentUser
{
    if (currentUser == nil)
    {
        return nil;
    }
    return currentUser;
}
</code></pre>

#### ViewController.h

<pre><code class="language-clike">@interface ViewController : UIViewController
{
    int savedException;
}
- (IBAction)getAllMessages:(id)sender;
</code></pre>

#### ViewController.m

Here we will make use of the get messages method from the Helper class and provide the failure block to throw an error.

<pre><code class="language-clike">#import "Helper.h"
#import "Message.h"

@interface ViewController ()
@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
	// Do any additional setup after loading the view, 
	// typically from a nib.
}

- (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (IBAction)getAllMessages:(id)sender
{
    NSArray *newArray = [Helper getAllMessagesForCurrentUser 
    		onFailure:^(int exc){

    	/**
    	* This is the block we are calling 
    	* inline to handle the error
    	**/

        [self throwError:exc];
    }];
    
    if (newArray != nil)
    {
        for (Message *message in newArray) {
            //Manipulate the Messages here
        };
    }
}

- (void)throwError:(int)exception
{
    savedException = exception;
    NSString *message = [[NSString alloc] init];
    NSString *title = [[NSString alloc] init];
    NSString *button = [NSString stringWithFormat:@"OK"];
    
    switch (exception) {
        case 0:
            title = @"An error occurred";
            message = @"Something went wrong, please try again";
            NSLog(@"Something went wrong saving to Stackmob");
            break;
        case 1:
            title = @"Not logged in";
            message = @"Please log in first";
            button = @"Login";
            break;
        default:
            title = @"An error occurred";
            message = @"Something went wrong, please try again";
            break;
    }
    
    /**
    * Fire an Alert View here and do some logic to 
    * show a login screen
    **/

}
</code></pre>

This has been a short introduction to Objective-C blocks. Of course, they *can* get incredibly complicated very quickly (if you desire), but I’m sure you can use this as a starting point to build on that knowledge. 

Take some time and leave a comment below with some feedback. I'd love to hear your opinions on this post, and on the implementation of blocks in Objective-C. 

[Developer API]: http://developer.apple.com/library/ios/documentation/cocoa/Conceptual/Blocks/Articles/bxOverview.html#//apple_ref/doc/uid/TP40007502-CH3-SW1
[Declaring iOS blocks]: http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/Blocks/Articles/bxDeclaringCreating.html