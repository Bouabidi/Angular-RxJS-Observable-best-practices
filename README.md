https://medium.com/grensesnittet/5-helpful-rxjs-solutions-d34f7c2f1cd9
# Angular-RxJS-Observable-best-practices
## Angular + Observable 

The Observable belongs to RxJS library. To perform asynchronous programming in Angular application, we can use either Observable or Promise. When we send and receive data over HTTP, we need to deal it asynchronously because fetching data over HTTP may take time.
The actual HTTP request hits the server only when the Observable instance is subscribed in TypeScript class or we use async pipe in HTML template.

 Some operators of RxJS library are subscribe, map, mergeMap, switchMap, exhaustMap, debounceTime, of, retry, catchError, throwError etc.
 
 To use RxJS library in Angular programming, we need to ensure that we have installed RxJS. In case we are using Angular CLI, it installs RxJS by default. We can ensure it by checking package.json. 
 
 ### Angular HttpClient and Observable
 
Angular HttpClient performs HTTP requests for the given URL. HttpClient works with Observable. Find some of the methods of HttpClient.
```javascript
get(url: string, options: {...}): Observable<any>
post(url: string, body: any | null, options: {...}): Observable<any>
put(url: string, body: any | null, options: {...}): Observable<any>
delete(url: string, options: {...}): Observable<any>
```
Look into return types of the above methods, each and every method of HttpClient returns instance of Observable. When we subscribe to the Observable instance, then actual HTTP operation is performed.

### Subscribe to Observable
subscribe() is a method of Observable class. subscribe is used to invoke Observable to execute HTTP request and to emit the result. If we have an Observable instance that fetches data over an HTTP then actual hit to server takes place only when we subscribe to Observable using subscribe method in TypeScript class or async pipe in HTML template.

1. Step-1
created a method in service to fetch data over HTTP.

```javascript
getBooksFromStore(): Observable<Book[]> {
   return this.http.get<Book[]>(this.bookUrl);
} 
```
2. Step-2: 
create a component property.

```javascript
softBooks: Book[] = [];

subscribe to the Observable result of getBooksFromStore()

getsoftBooks() {
   this.bookService.getBooksFromStore().subscribe(books => this.softBooks = books);
} 
```
3. Step-3:
display our array softBooks in HTML template using ngFor.

```javascript
<ul>
  <li *ngFor="let book of softBooks" >
    Id: {{book.id}}, Name: {{book.name}}, Category: {{book.category}}
  </li>
</ul> 
```
### Observable.pipe()

The Observable.pipe() pipe is used to combine RxJS operators into a chain

```javascript
getNumbers(): Observable<number> {
   return of(1, 2, 3, 4, 5, 6, 7);
}
calculateNumbers() {
  this.getNumbers().pipe(
    filter(n => n % 2 === 1),
    map(n => n + 10)
  )
  .subscribe(result => console.log(result));
} 
```
### Using map()
 map applies a given function to the value of its source Observable and then returns result as Observable. 
 
 ```javascript
 getBookName() {
   this.favBookName$ = this.bookService.getFavBookFromStore(101).pipe(
      map(book=> book.name)
   );  
} 
```
In the above code getFavBookFromStore() service method is returning Observable<Book>. Using map we have converted it into Observable<string>. The component property favBookName$ will now only emit book name. getFavBookFromStore() is defined as following.

```javascript
getFavBookFromStore(id: number): Observable<Book> {
   return this.http.get<Book>(this.bookUrl + "/" + id);
} 

favBookName$ can be displayed in HTML template as following.
<div *ngIf="favBookName$ | async as bookName">
  Name: {{bookName}}
</div> 

Find the code snippet to use map with subscribe().
getBookName() {
   this.bookService.getFavBookFromStore(101).pipe(
       map(book=> book.name)
   ).subscribe(name=> {
       this.favBookName = name;
       console.log(name);
   });
} 
```
### Using mergeMap()
mergeMap operator is basically a combination of two operators - merge and map. The map part lets you map a value from a source observable to an observable stream. Those streams are often referred to as inner streams. The merge part combines all inner observable streams returned from the map and concurrently emits all values from every input stream.

```javascript
getAllFavBooks() {
   this.myAllfavBooks$ = this.bookService.getFavBookFromStore(101)
      .pipe(mergeMap(book => this.bookService.getBooksByCategoryFromStore(book.category)));
} 
```
In the above code snippet, first we are fetching a favorite book using getFavBookFromStore method then by using this book's category we are fetching all books of same category using getBooksByCategoryFromStore() method. Our service methods are defined as following.

```javascript
getFavBookFromStore(id: number): Observable<Book> {
   return this.http.get<Book>(this.bookUrl + "/" + id);
}    
getBooksByCategoryFromStore(category: string): Observable<Book[]> {
   return this.http.get<Book[]>(this.bookUrl + "?category=" + category);
} 
```
myAllfavBooks$ can be displayed in HTML template as following.

```javascript
<ul>
  <li *ngFor="let book of myAllfavBooks$ | async" >
    Id: {{book.id}}, Name: {{book.name}}, Category: {{book.category}}
  </li>
</ul> 
```
switchMap operator is basically a combination of two operators - switchAll and map. The map part lets you map a value from a higher-order source observable to an inner observable stream. The switch part than works like switchAll - it subscribes to the most recently provided inner observable emitted

```javascript
searchSimilarBooks(id: number) {
   this.similarBooks$ = this.bookService.getFavBookFromStore(id).pipe(
	switchMap(book => {
		let category = book.category;
		return this.bookService.getBooksByCategoryFromStore(category);
	}),
	catchError(err => of([]))
   );
} 
```
### Using of()

The of() is a static method of Observable. The of is used to create a Observable instance using data passed as an argument.

Suppose we have a const value as following.

```javascript
const bookDetails: Book[] = [
    { id: 101, name: 'Angular by Krishna', category: 'Angular' },
    { id: 102, name: 'Core Java by Vishnu', category: 'Java' },
    { id: 103, name: 'NgRx by Rama', category: 'Angular' }
]; 
```
We can convert bookDetails into Observable by calling of(bookDetails) . We can create a method to get this result as following.

```javascript
getBooksFromStore(): Observable<Book[]> {
     return of(bookDetails);
}
```
### Using retry()
Some errors can be handled by just retrying request such as if errors are transient and unlikely to repeat. For example if we have slow network error and our request could not get success then if request is retried, there are the chances to make request successful
We can retry request automatically by using retry() operator. It accepts argument as number. Suppose we pass argument as retry(3) then the request will be retried for 3 times. The retry is imported from RxJS as following.

```javascript
this.bookService.getBooksFromStore().pipe(
     retry(3)
).subscribe(books => {
   this.softBooks = books;
},
(err: HttpErrorResponse) => {
   if (err.error instanceof Error) {
      //A client-side or network error occurred.
      console.log('An error occurred:', err.error.message);
   } else {
     //Backend returns unsuccessful response codes such as 404, 500 etc.
     console.log('Backend returned status code: ', err.status);
     console.log('Response body:', err.error);
   }
 }
); 
```
### Using Tap
Tap RxJs operator returns an observable that is identical to the source. It does not modify the stream in any way. Tap operator is useful for logging the value, debugging the stream for the correct values, or perform any other side effects.

We use the pipe to chain the tap operator, which just logs the values of the source observable into the console.
```javascript
of(1, 2, 3, 4, 5)
      .pipe(
        tap(val => {
          console.log("Tap " + val);
        })
      )
      .subscribe(val => console.log("at Subscriber " + val));	
```
The result of console is:
```javascript
Tap 1
at Subscriber 1
Tap 2
at Subscriber 2
Tap 3
at Subscriber 3
Tap 4
at Subscriber 4
Tap 5
at Subscriber 5	
```
One of the use cases for the tap operator is using it to debug the Observable for the correct values.
The map operator in the following example, adds 5 to the source observable. To debug it, we can add the two tap operators. One before and one after it and inspect the values.
```javascript
of(1, 2, 3, 4, 5)
      .pipe(
        tap(val => {
          console.log("before " +val);
        }),
        map(val => {
          return val + 5;
        }),
        tap(val => {
          console.log("after " +val);
        })
      )
      .subscribe(val => console.log(val));
 
 
 
**Console**
before 1
after 6
6
before 2
after 7
7
before 3
after 8
8	
```
	
We can also use the tap operator to log the error and complete callbacks as shown in the example below.
```javascript
 of(1, 2, 3, 4, 5)
      .pipe(
        tap(val => {
          console.log("Before " + val);
        }),
        map(val => {
          if (val == 3) {
            throw Error;
          }
          return val + 5;
        }),
        tap(
          val => {
            console.log("After " + val);
          },
          err => {
            console.log("Tap Error");
            console.log(err);
          },
          () => {
            console.log("Tap Complete");
          }
        )
      )
      .subscribe(val => console.log(val));

***Console ***
Before 1
After 6
6
Before 2
After 7
7
Before 3
Tap Error
 ƒ Error()
 
ERROR
 ƒ Error()
	
```
## Best Practices
### Pipeable operators
```javascript
// BAD: This is the old way and should be avoided (patch operators)
// as we can see the operators (filter, map) are part of the
// Observable prototype
import 'rxjs/add/operator/filter';
import 'rxjs/add/operator/map';
const new$ = Observable.interval$
    .filter(v => v % 2 === 0)
    .map(v => v * 2);

// GOOD: This is the new and improved way (lettable operators)
// we just use the pipe operator where we pass operators that
// we can import from 'rxjs/operators'
import {filter, map} from 'rxjs/operators';
const new$ = interval$
    .pipe(
        filter(v => v % 2 === 0),
        map(v => v *2)
    )	
```
### Marble diagrams
A cool way of visualising streams but it’s hard to put those marble-diagrams in our code right?! There is an ASCII variant of these marble-diagrams that we can use to describe and document our complex streams and how they interact with each other.
To check a real example and how we might document it:	
```javascript
const interval$ = interval(1000)            // 0--1--2--3--4--5--6...
const new$ = interval$
    .pipe(
        skip(1),                            // ---1--2--3--4--5--6...
        take(5),                            // ---1--2--3--4--5|
        filter(v => v % 2 === 0),           // ------2-----4---|
        map(v => v + 1)                     // ------3-----5---|
    )
```
### Handling of multiple HTTP requests
#### Scenario 1: Sequential requests. Using data from previous requests in next requests.
We are often required to make requests based on the response from earlier requests. This is true in the scenario below.
1. We first retrieve a post based on an id.
2. Using the post response from the first call, we then want to retrieve the user of the post.
3. After we have the user we want to get all the posts of the retrieved user.
```javascript
this.postService.loadPost(postId).subscribe(post => {
   this.userService.loadUser(post.userId).subscribe(user => {
     this.postService.loadPosts(user.id).subscribe(posts => {
 ...
	
```
This can be easily simplified with the operator switchMap. The functionality of switchMap lies in its name. After the first observable emits it then subscribes to an inner observable.
	
This is perfect for our scenario. We want to use our first observable of posts and map the result to a new observable that loads users.
The operator will take the outer observable of a post and when it emits, switch to the inner observable of ‘user’ and map the result of the outer observable to our inner observable.	
	
```javascript
this.postService.loadPost(postId).pipe(
     // map our observable of a post to a new observable of user
     switchMap(post => this.userService.loadUser(post.userId)),

     // map our previous observable of a user to a new observable of all the users posts
     switchMap(user => this.userService.loadPosts(user.id)),
   )
   .subscribe(posts => console.log(posts));
```
#### Scenario 2: Making parallel requests.
Sometimes we don’t care about the order in which multiple requests finish. We may just want to collect them once they are all done. Say we only have a userId. We want to get all the posts the user has made and metadata about the same user.
As each of these two requests are independent of each other the order in which they finish is not important. We do however want to wait until we have the data from both. Often when loading data we also want to show some sort of loading-indicator.
Below is a naive solution where we patch an object to indicate that we have the data.	
```javascript
let userWithPosts = {
  user: null,
  posts: null
};

this.postService.loadPosts(userId).subscribe(posts => {
  userWithPosts.posts = posts;
});

this.userService.loadUser(userId).subscribe(user => {
  userWithPosts.user = user;
});

// In HTML
<p *ngIf=”!(userWithPosts.user || userWithPosts.posts)”>Loading...</p>
<div *ngIf=”userWithPosts.user && userWithPosts.posts”>... do stuff with the data</div>
```
With RxJs this can be solved in just a few lines of code with the operator forkJoin.
```javascript
let userWithPosts;

forkJoin({
  posts: this.postService.loadPosts(userId)
  user: this.userService.loadUser(userId)
}).subscribe(response => {
  this.userWithPosts = response;
 })

 // In HTML
 <p *ngIf=”!userWithPosts”>Loading...</p>
 <div *ngIf=”userWithPosts”>... do stuff with the data</div>	
```
### Combining multiple streams into one for a search feature
Implement a smart search where requests were composed of multiple sources such as:
1. A searchTerm$ coming from an input field
2. A set of metadataFilters$ (ranges, checkboxes)
3. A set of polygons$ that could be drawn on a map
4. The currentResultPage$ coming from a result page with pagination buttons
The source data came from different components and so we needed an easy readable way of combining them in one handy stream.
	
Solution: combineLatest
The combineLatest function combines an array of streams into one and emits them all whenever one of them changes.
```javascript
combineLatest([
  this.searchTerm$,
  this.metadataFilters$,
  this.polygons$,
  this.currentResultPage$
]).subscribe(requestBody => this.search(requestBody))	
```
We can improve our solution further with a new operator debounceTime and our friend from our first scenario switchMap. Remember from before that another feature of switchMap is that it will cancel the previous request if a new one is made before the current one has completed. This means we always get the result from the request that was made last.	
