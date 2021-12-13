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
