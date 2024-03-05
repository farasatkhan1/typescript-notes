`keyof typeof routes` can be used to extract the keys from the object. `(typeof routes)[keyof typeof routes]` can be used to extract all the values from the object. We can access item at specific index using `(typeof routes)['admin']` will return the value of the admin key which will be `/admin`. We can use the union operator to pass in multiple Keys like `(typeof routes)['admin' | 'users']` will now return `'/admin' | '/users'`.

```typescript
const routes = {
	home: "/",
	admin: "/admin",
	users: "/users",
	newUser: "/users/new"
} as const;

type Route = (typeof routes)[keyof typeof routes]

const goToRoute = (route: Route) => {};

goToRoute(routes.admin)
```


`Object.freeze()` works at runtime while `as const` works at compile time. Both of them prevents the object from manipulating. `Object.freeze()` only works at the top level so if we have deeply nested objects then it will not work properly. On the other hand `as const` prevents the object, including the deeply nested objects from manipulating.

```typescript
const routes = {
	home: "/",
	admin: "/admin",
	users: "/users",
	newUser: "/users/new"
} as const;

const routesAlt = Object.freeze({
	home: "/",
	admin: "/admin",
	users: "/users",
	newUser: "/users/new"
})
```


# any vs unknown vs never

The `any` operator disables any typechecking on the values we assigned the type any to. We can treat the `val` variable as a string or number or anything we want. Typescript will not throw any error if we misuse it and thus we should avoid using the `any` operator. Let say we assigned a number to variable and later try to use functions when can only be applied on the String variable and it will not throw an error during compile time.

```typescript
let val: any = 1;
```

`unknown` operator should instead be preferred over the `any` operator. `unknown` operator let the function accepts different type of values. If we use the `unknown` operator then we would need to narrow it to use it accordinly.

```typescript
let val: unknown = 1;

if (typeof val === 'number') val++;
```

The `UnknownFunc` accepts values of different types and then treat it differently according to the type of the value.

```typescript
function UnknownFunc(val: unknown) {
    if (typeof val === 'number') return val++;
    if (typeof val === 'string') return val.toUpperCase();

    return null;
}

console.log(UnknownFunc(1));
console.log(UnknownFunc('hello world'));
```

If value is an object then we need to do some checking before using it.

```typescript
function nameToUppercase(val: unknown) {
    if (
        val &&
        typeof val === 'object' &&
        'name' in val &&
        typeof val.name === 'string'
    ) return val.name.toUpperCase();
   
    return null;
}

const obj: Record<string, string | number> = {
    name: 'Ben',
    age: 25
};

// Output: BEN
console.log(nameToUppercaseAlt(obj));
```

```typescript
function nameToUpperCaseAlt(val: unknown) {
    if (
        val &&
        typeof val === 'object' &&
        'name' in val &&
        typeof (val as { name: string }).name === 'string'
    ) return (val as { name: string }).name.toUpperCase();
	
    return null;
}

const obj: Record<string, string | number> = {
    name: 'Ben',
    age: 25
};

// Output: BEN
console.log(nameToUpperCase(obj)); 
```

```typescript
function ageIncrementor(val: unknown) {
    if (
	    val &&
        typeof val === 'object' &&
        'age' in val &&
        typeof val.age === 'number'
    ) {
        return val.age += 1;
    }
    return null;
}

const obj: Record<string, string | number> = {
    name: 'Ben',
    age: 25
};

// Output: 26
console.log(ageIncrementor(obj));
```


```typescript
function ageIncrementorAlt(val: unknown) {
    if (
        val &&
        typeof val === 'object' &&
        'age' in val &&
        typeof (val as {age: number}).age === 'number'
    ) return (val as { age: number }).age += 1;

    return null;
}

const obj: Record<string, string | number> = {
    name: 'Ben',
    age: 25
};

// Output: 26
console.log(ageIncrementor(obj));
```

The `never` type represents the type of values which never occur. It is useful where a function never returns or when it throws an error which halts the execution. It is useful to use in function which never returns a value.

```typescript
function throwSomeError (message: string): never {
	throw new Error(message);
}
```

It can also be useful in cases where we exhaust all the possibilities. In a type guard, if typescript is unable to narrow down the type to a specific set of possibilities, we can use the `never` type to signal that remaining possibilities are impossible. Once we exhaust all the possibilities the type of the `user` will be never and if there are some possibilities left then `_unreachableCode` will throw an error.

```typescript
type User = 'user' | 'admin';

function login(user: User) {
    switch (user) {
        case 'admin':
            return 'admin';
        case 'user':
            return 'user';
        default:
            const _unreachableCode: never = user;
            return false;
    }
}
```

