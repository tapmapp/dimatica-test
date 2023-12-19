# NodeJs

## Exercice: Is there a problem? _(1 points)_

```javascript
// Call web service and return count user, (got is library to call url)
async function getCountUsers() {
  return {
    total: await got.get("https://my-webservice.moveecar.com/users/count"),
  };
}

// Add total from service with 20
async function computeResult() {
  const result = await getCountUsers();
  return result.total + 20;
}

Missing await

```

## Exercice: Is there a problem? _(2 points)_

```javascript
// Call web service and return total vehicles, (got is library to call url)
async function getTotalVehicles() {
  return await got.get("https://my-webservice.moveecar.com/vehicles/total");
}

async function getPlurial() {
  let total;
  await getTotalVehicles().then((r) => (total = r));
  if (total <= 0) {
    return "none";
  }
  if (total <= 10) {
    return "few";
  }
  return "many";
}

Missing async and await

```

## Exercice: Unit test _(2 points)_

Write unit tests in jest for the function below in typescript

```typescript
import { expect, test } from '@jest/globals';

function getCapitalizeFirstWord(name: string): string {
  if (name == null) {
    throw new Error('Failed to capitalize first word with null');
  }
  if (!name) {
    return name;
  }
  return name.split(' ').map(
    n => n.length > 1 ? (n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()) : n
  ).join(' ');
}

test('1. test', async function () {
    ...
});
```

# Angular

## Exercice: Is there a problem and improve the code _(5 points)_

```typescript


export interface User {
  email: string;
}

@Injectable()
export class UserService {

    constructor(private httpClient: HttpClient) {}

    public findUsers(query: string): Observable<User[]> {
        return this.httpClient.get()
    }

}

@Component({
  selector: "app-users",
  template: `
    <input
      type="text"
      [(ngModel)]="query"
      (ngModelChange)="querySubject.next($event)"
    />
    <div *ngFor="let user of users">
      {{ user.email }}
    </div>
  `,
})
export class AppUsers implements OnInit {
  query = "";
  querySubject = new Subject<string>();

  users: { email: string }[] = [];

  constructor(private userService: UserService) {}

  ngOnInit(): void {
        concat(of(this.query), this.querySubject.asObservable())
      .pipe(
        concatMap((q) =>
          timer(0, 60000).pipe(concatMap(async () => await this.findUsers(q)))
        )
      )
      .subscribe({
        next: (res) => (this.users = res),
      });
  }
}


Pipe expects an OperatorFunction, not a service, you need to wrap it into a concatMap to make it work.
Also, if you are planning to make a request you need to use async await

```

## Exercice: Improve performance _(5 points)_

```typescript
@Component({
  selector: "app-users",
  template: `
    <div *ngFor="let user of users | async">
      {{ getCapitalizeFirstWord(user.name) }}
    </div>
  `,
})
export class AppUsers {
  @Input()
  users: { name: string }[];

  constructor() {}

  public getCapitalizeFirstWord(name: string): Promise<string> {
    return new Promise((resolve) => resolve(name.split(" ").map((n) => n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()).join(" "));
  }
}
```

## Exercice: Forms _(8 points)_

Complete and modify `AppUserForm` class to use Angular Reactive Forms. Add a button to submit.

The form should return data in this format

```typescript
{
  email: string; // mandatory, must be a email
  name: string; // mandatory, max 128 characters
  birthday?: Date; // Not mandatory, must be less than today
  address: { // mandatory
    zip: number; // mandatory
    city: string; // mandatory, must contains only alpha uppercase and lower and space
  };
}
```

```typescript
@Component({
  selector: "app-user-form",
  template: `
    <form [formGroup]="userForm" (ngSubmit)="doSubmit()">
      <input formControlName="email" type="text" placeholder="email" />
      <input formControlName="name" type="text" placeholder="name" />
      <input formControlName="birthday" type="date" placeholder="birthday" />
      <input formControlName="zip" type="number" placeholder="zip" />
      <input formControlName="city" type="text" placeholder="city" />
      <button [disabled]="!userForm.valid">Save</button>
    </form>
  `,
})
export class AppUserForm implements OnInit {
  @Output() event = new EventEmitter<{
    email: string;
    name: string;
    birthday: Date;
    address: { zip: number; city: string };
  }>();
  public userForm: FormGroup;

  constructor(private formBuilder: FormBuilder) {}

  private initForm(): void {
    this.userForm = this.formBuilder.group({
      email: ["", [Validators.required]],
      name: ["", [Validators.required]],
      birthday: ["", [Validators.required]],
      zip: ["", [Validators.required]],
      city: ["", [Validators.required]],
    });
  }

  doSubmit(): void {
    this.event.emit({
      email: this.userForm.get("email").value,
      email: this.userForm.get("name").value,
      email: this.userForm.get("birthday").value,
      email: this.userForm.get("zip").value,
      email: this.userForm.get("city").value,
    });
  }

  ngOnInit(): void {
    this.initForm();
  }
}
```

# CSS & Bootstrap

## Exercice: Card _(5 points)_

![image](uploads/0388377207d10f8732e1d64623a255b6/image.png)

# MongoDb

## Exercice: MongoDb request _(3 points)_

MongoDb collection `users` with schema

```typescript
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
  }
```

Complete the query, you have a variable that contains a piece of text to search for. Search by exact email, starts with first or last name and only users logged in for 6 months

```typescript
function search(searchString: string) {
  return db.collections("users").find({
    email: searchString,
    $or: [
      { first_name: { $regex: searchString, $options: "i" } },
      { last_name: { $regex: searchString, $options: "i" } },
    ],
    { last_connection_date : { $gte : new Date(new Date().getTime() - (6 * 30 * 24 * 60 * 60 * 1000) }}
  });
}
```

What should be added to the collection so that the query is not slow?

I think you should set the email as unique in order to index it.

## Exercice: MongoDb aggregate _(5 points)_

MongoDb collection `users` with schema

```typescript
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
  }
```

Complete the aggregation so that it sends user emails by role ({\_id: 'role', users: [email,...]})

```typescript
db.collections('users').aggregate({
    $group: {
      _id: "role",
      users:
    }
  });
```

## Exercice: MongoDb update _(5 points)_

MongoDb collection `users` with schema

```typescript
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
    addresses: {
        zip: number;
        city: string;
    }[]:
  }
```

Update document `ObjectId("5cd96d3ed5d3e20029627d4a")`, modify only `last_connection_date` with current date

```typescript
db.collections('users').updateOne({ _id: ObjectId("5cd96d3ed5d3e20029627d4a"), { $set: last_connection_date: new Date() } });
```

Update document `ObjectId("5cd96d3ed5d3e20029627d4a")`, add a role `admin`

```typescript
db.collections('users').updateOne({_id: ObjectId("5cd96d3ed5d3e20029627d4a"), { $push: roles: "admin" });
```

Update document `ObjectId("5cd96d3ed5d3e20029627d4a")`, modify addresses with zip `75001` and replace city with `Paris 1`

```typescript
db.collections('users').updateOne({ _id: ObjectId("5cd96d3ed5d3e20029627d4a"), { addresses: { $elemMatch: { zip: "75001" }} }, {$set: "addresses.$.zip": "Paris 1"});
```
