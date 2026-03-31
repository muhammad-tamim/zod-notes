<h1 align="center">Zod Notes</h1>

- [Introduction:](#introduction)
  - [Installation:](#installation)
  - [Basic Usage:](#basic-usage)
    - [Defining a schema:](#defining-a-schema)
    - [Parsing data:](#parsing-data)
    - [Inferring types:](#inferring-types)
    - [Handling errors:](#handling-errors)
      - [Customizing errors:](#customizing-errors)
- [Defining schemas:](#defining-schemas)
  - [Primitives Types:](#primitives-types)
  - [Coercion:](#coercion)
  - [Literals:](#literals)
  - [Strings:](#strings)
  - [String formats:](#string-formats)
  - [Template literals:](#template-literals)
  - [Numbers:](#numbers)
  - [Integers:](#integers)
  - [BigInts:](#bigints)
  - [Booleans:](#booleans)
  - [Dates:](#dates)
  - [Enums:](#enums)
    - [.enum:](#enum)
    - [.exclude():](#exclude)
    - [.extract():](#extract)
  - [Optionals:](#optionals)
  - [Objects:](#objects)
    - [z.strictObject:](#zstrictobject)
    - [.catchall():](#catchall)
    - [.shape:](#shape)
    - [.extend():](#extend)
    - [.safeExtend()](#safeextend)
    - [.partial():](#partial)
    - [.required():](#required)
  - [Arrays:](#arrays)
  - [Tuples:](#tuples)
  - [Unions:](#unions)
  - [Exclusive unions (XOR):](#exclusive-unions-xor)
  - [Intersections:](#intersections)
  - [Refinements:](#refinements)
  - [Pipes:](#pipes)
  - [Transforms:](#transforms)
    - [.transform():](#transform)
    - [Defaults:](#defaults)
  - [Catch:](#catch)


# Introduction: 
Zod is a TypeScript-first schema validation library used to define the shapes, validate, and infer types automatically of our data at runtime. TypeScript alone can just do compile time safety and just give us editor level runtime error suggestions, but Zod give us runtime safety + compile time safety (with ts). Zod is often used in scenarios like validating API requests, form inputs, or any other data that needs to be checked for correctness before being processed further. 

Without zod, blow code run perfectly and we won't get any error until we compile it with tsc.

```ts
type Player = {
    username: string;
    xp: number;
}

const input: Player = { username: "billie", xp: '100' };

console.log(input);
```

But with zod, now we get editor label error by ts + runtime error by zod: 

```ts
import * as z from "zod";

const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
});

type PlayerType = z.infer<typeof PlayerSchema>;

const input: PlayerType = { username: "billie", xp: '100' };

const playerData = PlayerSchema.parse(input);

console.log(playerData);
```

```js
ZodError: [
  {
    "expected": "number",
    "code": "invalid_type",
    "path": [
      "xp"
    ],
    "message": "Invalid input: expected number, received string"
  }
]
```

## Installation: 

```bash
npm install zod
```

## Basic Usage:

### Defining a schema:
A schema is simply a blueprint that describes what valid data should look like.

```ts
import * as z from "zod"; 
 
const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
});
```

### Parsing data:
use `.parse` to validate an schema input. If it's valid, Zod returns a strongly-typed deep clone of the input.

```ts
import * as z from "zod";

const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
});

const input = { username: "billie", xp: '100' };

const playerData = PlayerSchema.parse(input);

console.log(playerData);
```

Note: now zod only give us runtime error, but if we want to get editor label error by ts, if we want editor label error by ts, we need to  use z.infer<> to extract the inferred type from our schema.


**Note:** If our schema uses certain asynchronous APIs like async refinements or transforms, we'll need to use the .parseAsync() method instead. 

```ts
await PlayerSchema.parseAsync(input); 
```

### Inferring types: 
Zod infers static type from our schema definitions as type alias by default. we can extract this type with the z.infer<> utility and use it as a normal type alias.

**Note:** For interface zod doesn't support to inter types, but type alisa are supported.

```ts
import * as z from "zod";

const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
});

type PlayerType = z.infer<typeof PlayerSchema>

const input: PlayerType = { username: "billie", xp: '100' };

const playerData = PlayerSchema.parse(input);

console.log(playerData);
```

Now we get editor label error by ts + runtime error by zod.

Note: Zod infers static type form our schema definitions as type alias by default, to prove it we can see the below example: 

```ts
import * as z from "zod";

const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
});


const input = { username: "billie", xp: '100' };

const playerData = PlayerSchema.parse(input);

console.log(playerData);
```

Now hover the input variable on vs code, then we can see the type of input is: 

```
const input: {
    username: string;
    xp: string;
}
```

### Handling errors:

```tsx
import * as z from "zod";

const App = () => {

  const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
  });

  type PlayerType = z.infer<typeof PlayerSchema>

  const input: PlayerType = { username: "billie", xp: '100' };

  const playerData = PlayerSchema.parse(input);

  console.log(playerData);
  return (
    <div>
    </div>
  );
};

export default App;
```

To prevent our application from breaking, we can use try/catch block for with .parse() to catch the error and handle it gracefully, and if we don't want to use try/catch block, we can use .safeParse().


```ts
import * as z from "zod";

const App = () => {
  const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
  });

  type PlayerType = z.infer<typeof PlayerSchema>

  const input: PlayerType = { username: "billie", xp: '100' };

  try {
    const playerData = PlayerSchema.parse(input);
    console.log(playerData);
  }
  catch (error) {
    if (error instanceof z.ZodError) {
      console.log(error)
      console.log(error.issues)
      console.log(error.issues[0])
      console.log(error.issues[0].message)
    }
  }
  return (
    <div>
      hi
    </div>
  );
};

export default App;
```

here, 
- error: gives the error as ZodError instance, but we can't show it directly by ui for users.
- error.issues: gives the error as array of objects, each object contains the details of the error.
- error.issues[0]:  gives the first error object here we can find all commonly used important errors

![alt text](./assets/images/handling-errors.png) 


for .safeParse(): 

```tsx
import * as z from "zod";

const App = () => {
  const PlayerSchema = z.object({
    username: z.string(),
    xp: z.number()
  });

  type PlayerType = z.infer<typeof PlayerSchema>

  const input: PlayerType = { username: "billie", xp: '100' };

  const result = PlayerSchema.safeParse(input);

  if (!result.success) {
    console.log(result.error)
    console.log(result.error.issues)
    console.log(result.error.issues[0])
    console.log(result.error.issues[0].message)
  } else {
    console.log(result.data)
  }
  return (
    <div>
      hi
    </div>
  );
};

export default App;
```

Note: If our schema uses certain asynchronous APIs like async refinements or transforms, we'll need to use the .safeParseAsync() method instead.

#### Customizing errors: 
If we don't like the default error message, Virtually every Zod API accepts an optional error message. so using that we can customize the error message for each schema: 

```ts
z.string("Not a string!");
```

This custom error will show up as the message property of any validation issues that originate from this schema: 

```ts
z.string("Not a string!").parse(12);
// ❌ throws ZodError {
//   issues: [
//     {
//       expected: 'string',
//       code: 'invalid_type',
//       path: [],
//       message: 'Not a string!'   <-- 👀 custom error message
//     }
//   ]
// }
```


All z functions and schema methods accept custom errors: 

```ts
z.string("Bad!");
z.string().min(5, "Too short!");
z.uuid("Bad UUID!");
z.iso.date("Bad date!");
z.array(z.string(), "Not an array!");
z.array(z.string()).min(5, "Too few items!");
z.set(z.string(), "Bad set!");
```

If you prefer, you can pass a params object with an error parameter instead: 

```ts
z.string({ error: "Bad!" });
z.string().min(5, { error: "Too short!" });
z.uuid({ error: "Bad UUID!" });
z.iso.date({ error: "Bad date!" });
z.array(z.string(), { error: "Bad array!" });
z.array(z.string()).min(5, { error: "Too few items!" });
z.set(z.string(), { error: "Bad set!" });
```



# Defining schemas: 

## Primitives Types: 

```ts
z.string();
z.number();
z.bigint();
z.boolean();
z.symbol();
z.undefined();
z.null();
```

## Coercion: 
To coerce input data to the appropriate type, use z.coerce instead:

```ts
z.coerce.string();    // String(input)
z.coerce.number();    // Number(input)
z.coerce.boolean();   // Boolean(input)
z.coerce.bigint();    // BigInt(input)
```

Note: By default the input type of any z.coerce schema is unknown. In some cases, it may be preferable for the input type to be more specific. You can specify the input type with a generic parameter.

```ts
const A = z.coerce.number();
type AInput = z.input<typeof A>; // => unknown
 
const B = z.coerce.number<number>();
type BInput = z.input<typeof B>; // => number
```

## Literals: 

```ts
const tuna = z.literal("tuna");
const twelve = z.literal(12);
const twobig = z.literal(2n);
const tru = z.literal(true);
```

To allow multiple literal values:

```ts
const colors = z.literal(["red", "green", "blue"]);
 
colors.parse("green"); // ✅
colors.parse("yellow"); // ❌
```


## Strings: 
Zod provides a handful of built-in string validation and transform APIs. To perform some common string validations:

```ts
z.string().max(5);
z.string().min(5);
z.string().length(5);
z.string().regex(/^[a-z]+$/);
z.string().startsWith("aaa");
z.string().endsWith("zzz");
z.string().includes("---");
z.string().uppercase();
z.string().lowercase();
```

To perform some simple string transforms:

```ts
z.string().trim(); // trim whitespace
z.string().toLowerCase(); // toLowerCase
z.string().toUpperCase(); // toUpperCase
z.string().normalize(); // normalize unicode characters
```

## String formats: 
To validate against some common string formats:

```
z.email();
z.uuid();
z.url();
z.httpUrl();       // http or https URLs only
z.hostname();
z.emoji();         // validates a single emoji character
z.base64();
z.base64url();
z.hex();
z.jwt();
z.nanoid();
z.cuid();
z.cuid2();
z.ulid();
z.ipv4();
z.ipv6();
z.mac();
z.cidrv4();        // ipv4 CIDR block
z.cidrv6();        // ipv6 CIDR block
z.hash("sha256");  // or "sha1", "sha384", "sha512", "md5"
z.iso.date();
z.iso.time();
z.iso.datetime();
z.iso.duration();
```

## Template literals: 
```ts
z.templateLiteral([ "hello, ", z.string(), "!" ]);
// `hello, ${string}!`

z.templateLiteral([ "hi there" ]);
// `hi there`
 
z.templateLiteral([ "email: ", z.string() ]);
// `email: ${string}`
 
z.templateLiteral([ "high", z.literal(5) ]);
// `high5`
 
z.templateLiteral([ z.nullable(z.literal("grassy")) ]);
// `grassy` | `null`
 
z.templateLiteral([ z.number(), z.enum(["px", "em", "rem"]) ]);
// `${number}px` | `${number}em` | `${number}rem`
```

## Numbers: 
```ts
const schema = z.number();
 
schema.parse(3.14);      // ✅
schema.parse(NaN);       // ❌
schema.parse(Infinity);  // ❌
```

Zod implements a handful of number-specific validations:

```ts
z.number().gt(5);
z.number().gte(5);                     // alias .min(5)
z.number().lt(5);
z.number().lte(5);                     // alias .max(5)
z.number().positive();                 // alias .gt(0)
z.number().nonnegative();    
z.number().negative(); 
z.number().nonpositive(); 
z.number().multipleOf(5);              // alias .step(5)
```

## Integers: 

```ts
z.int();     // restricts to safe integer range
z.int32();   // restrict to int32 range
```

## BigInts: 

```ts
z.bigint();
```

Zod includes a handful of bigint-specific validations: 

```ts
z.bigint().gt(5n);
z.bigint().gte(5n);                    // alias `.min(5n)`
z.bigint().lt(5n);
z.bigint().lte(5n);                    // alias `.max(5n)`
z.bigint().positive();                 // alias `.gt(0n)`
z.bigint().nonnegative(); 
z.bigint().negative(); 
z.bigint().nonpositive(); 
z.bigint().multipleOf(5n);             // alias `.step(5n)`
```

## Booleans: 

```ts
z.boolean().parse(true); // => true
z.boolean().parse(false); // => false
```

## Dates: 
```ts
z.date().safeParse(new Date()); // success: true
z.date().safeParse("2022-01-12T06:15:00.000Z"); // success: false
```

Zod provides a handful of date-specific validations:

```ts
z.date().min(new Date("1900-01-01"), { error: "Too old!" });
z.date().max(new Date(), { error: "Too young!" });
```

## Enums: 
Use z.enum to validate inputs against a fixed set of allowable string values: 

```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);
 
FishEnum.parse("Salmon"); // => "Salmon"
FishEnum.parse("Swordfish"); // => ❌
```

Note: If we declare our string array as a variable, Zod won't be able to properly infer the exact values of each element.

```ts
const fish = ["Salmon", "Tuna", "Trout"];
 
const FishEnum = z.enum(fish);
type FishEnum = z.infer<typeof FishEnum>; // string
```

To fix this we need to use as const assertion: 

```ts
const fish = ["Salmon", "Tuna", "Trout"] as const;
 
const FishEnum = z.enum(fish);
type FishEnum = z.infer<typeof FishEnum>; // "Salmon" | "Tuna" | "Trout"
```

Enum-like object literals ({ [key: string]: string | number }) are also supported: 

```ts
const Fish = {
  Salmon: 0,
  Tuna: 1
} as const
 
const FishEnum = z.enum(Fish)
FishEnum.parse(Fish.Salmon); // => ✅
FishEnum.parse(0); // => ✅
FishEnum.parse(2); // => ❌
```

### .enum: 
To extract the schema's values as an enum-like object:

```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);
 
FishEnum.enum;
// => { Salmon: "Salmon", Tuna: "Tuna", Trout: "Trout" }
```

### .exclude(): 
To create a new enum schema, excluding certain values:
```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);
const TunaOnly = FishEnum.exclude(["Salmon", "Trout"]);
```

### .extract(): 
To create a new enum schema, extracting certain values:

```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);
const SalmonAndTroutOnly = FishEnum.extract(["Salmon", "Trout"]);
```

## Optionals: 
To make a schema optional (that is, to allow undefined inputs).

```ts
z.optional(z.literal("yoda")); // or z.literal("yoda").optional()
z.optional(z.string()); // or z.string().optional()
```

## Objects: 
To define an object type:

```ts
// all properties are required by default
const Person = z.object({
  name: z.string(),
  age: z.number(),
});

type Person = z.infer<typeof Person>;
// => { name: string; age: number; }
```

By default, all properties are required. To make certain properties optional:

```ts
const Dog = z.object({
  name: z.string(),
  age: z.number().optional(),
});
 
Dog.parse({ name: "Yeller" }); // ✅
```

By default, unrecognized keys are stripped from the parsed result:

```ts
Dog.parse({ name: "Yeller", extraKey: true });
// => { name: "Yeller" }
```

### z.strictObject: 
To define a strict schema that throws an error when unknown keys are found:

```ts
const StrictDog = z.strictObject({
  name: z.string(),
});
 
StrictDog.parse({ name: "Yeller", extraKey: true });
// ❌ throws
```

### .catchall(): 
To define a catchall schema that will be used to validate any unrecognized keys:

```ts
const DogWithStrings = z.object({
  name: z.string(),
  age: z.number().optional(),
}).catchall(z.string());
 
DogWithStrings.parse({ name: "Yeller", extraKey: "extraValue" }); // ✅
DogWithStrings.parse({ name: "Yeller", extraKey: 42 }); // ❌
```

### .shape: 
To access the internal schemas:

```ts
Dog.shape.name; // => string schema
Dog.shape.age; // => number schema
```

### .extend(): 
To add additional fields to an object schema:

```ts
const DogWithBreed = Dog.extend({
  breed: z.string(),
});
```

Note: This API can be used to overwrite existing fields! Be careful with this power! If the two schemas share keys, B will override A.

### .safeExtend()
The .safeExtend() method works similarly to .extend(), but it won't let you overwrite an existing property with a non-assignable schema. In other words, the result of .safeExtend() will have an inferred type that extends the original (in the TypeScript sense): 

```ts
z.object({ a: z.string() }).safeExtend({ a: z.string().min(5) }); // ✅
z.object({ a: z.string() }).safeExtend({ a: z.any() }); // ✅
z.object({ a: z.string() }).safeExtend({ a: z.number() });
//                                       ^  ❌ ZodNumber is not assignable 
```

### .partial(): 
To make all fields optional:

```ts
const PartialRecipe = Recipe.partial();
// { title?: string | undefined; description?: string | undefined; ingredients?: string[] | undefined }
```

To make certain properties optional:

```ts
const RecipeOptionalIngredients = Recipe.partial({
  ingredients: true,
});
// { title: string; description?: string | undefined; ingredients?: string[] | undefined }
```

### .required(): 

To make all properties required:

```ts
const RequiredRecipe = Recipe.required();
// { title: string; description: string; ingredients: string[] }
```

To make certain properties required:
```ts
const RecipeRequiredDescription = Recipe.required({description: true});
// { title: string; description: string; ingredients: string[] }
```

## Arrays: 

```ts
const stringArray = z.array(z.string()); // or z.string().array()
```
To access the inner schema for an element of the array: 

```ts
stringArray.unwrap(); // => string schema
```

Zod implements a number of array-specific validations:

```ts
z.array(z.string()).min(5); // must contain 5 or more items
z.array(z.string()).max(5); // must contain 5 or fewer items
z.array(z.string()).length(5); // must contain 5 items exactly
```

## Tuples: 

```ts
const MyTuple = z.tuple([
  z.string(),
  z.number(),
  z.boolean()
]);
 
type MyTuple = z.infer<typeof MyTuple>;
// [string, number, boolean]
```

## Unions: 

```ts
const stringOrNumber = z.union([z.string(), z.number()]);
// string | number
 
stringOrNumber.parse("foo"); // passes
stringOrNumber.parse(14); // passes
```

## Exclusive unions (XOR): 
An exclusive union (XOR) is a union where exactly one option must match. Unlike regular unions that succeed when any option matches, z.xor() fails if zero options match OR if multiple options match: 

```ts
const schema = z.xor([z.string(), z.number()]);
 
schema.parse("hello"); // ✅ passes
schema.parse(42);      // ✅ passes
schema.parse(true);    // ❌ fails (zero matches)
```

## Intersections: 
Intersection types (A & B) represent a logical "AND": 

```ts
const a = z.union([z.number(), z.string()]);
const b = z.union([z.number(), z.boolean()]);
const c = z.intersection(a, b);
 
type c = z.infer<typeof c>; // => number
```

This can be useful for intersecting two object types: 

```ts
const Person = z.object({ name: z.string() });
type Person = z.infer<typeof Person>;
 
const Employee = z.object({ role: z.string() });
type Employee = z.infer<typeof Employee>;
 
const EmployedPerson = z.intersection(Person, Employee);
type EmployedPerson = z.infer<typeof EmployedPerson>;
// Person & Employee
```

## Refinements: 
Every Zod schema stores an array of refinements. Refinements are a way to perform custom validation that Zod doesn't provide a native API for: 

```ts
const myString = z.string().refine((val) => val.length <= 255);
```

To customize the error message:

```ts
const myString = z.string().refine((val) => val.length > 8, { 
  error: "Too short!" 
});
```

By default, validation issues from checks are considered continuable: 

```ts
const myString = z.string()
  .refine((val) => val.length > 8, { error: "Too short!" })
  .refine((val) => val === val.toLowerCase(), { error: "Must be lowercase" });
  
 
const result = myString.safeParse("OH NO");
result.error?.issues;
/* [
  { "code": "custom", "message": "Too short!" },
  { "code": "custom", "message": "Must be lowercase" }
] */
```

To mark a particular refinement as non-continuable, use the abort parameter:

```ts
const myString = z.string()
  .refine((val) => val.length > 8, { error: "Too short!", abort: true })
  .refine((val) => val === val.toLowerCase(), { error: "Must be lowercase", abort: true });
 
 
const result = myString.safeParse("OH NO");
result.error?.issues;
// => [{ "code": "custom", "message": "Too short!" }]
```

## Pipes: 
Schemas can be chained together into "pipes". Pipes are primarily useful when used in conjunction with Transforms:

```ts
const stringToLength = z.string().pipe(z.transform(val => val.length));
 
stringToLength.parse("hello"); // => 5
```

## Transforms:
Transforms are a special kind of schema that perform a unidirectional transformation. Instead of validating input, they accept anything and perform some transformation on the data. To define a transform:

```ts
const castToString = z.transform((val) => String(val));
 
castToString.parse("asdf"); // => "asdf"
castToString.parse(123); // => "123"
castToString.parse(true); // => "true"
```

Most commonly, transforms are used in conjunction with Pipes. This combination is useful for performing some initial validation, then transforming the parsed data into another form: 

```ts
const stringToLength = z.string().pipe(z.transform(val => val.length));
 
stringToLength.parse("hello"); // => 5
```

### .transform(): 
Piping some schema into a transform is a common pattern, so Zod provides a convenience .transform() method:

```ts
const stringToLength = z.string().transform(val => val.length); 
```

### Defaults: 
To set a default value for a schema:

```ts
const defaultTuna = z.string().default("tuna");
 
defaultTuna.parse(undefined); // => "tuna"
```

```ts
const randomDefault = z.number().default(Math.random);
 
randomDefault.parse(undefined);    // => 0.4413456736055323
randomDefault.parse(undefined);    // => 0.1871840107401901
randomDefault.parse(undefined);    // => 0.7223408162401552
```

## Catch: 
Use .catch() to define a fallback value to be returned in the event of a validation error:

```ts
const numberWithCatch = z.number().catch(42);
 
numberWithCatch.parse(5); // => 5
numberWithCatch.parse("tuna"); // => 42
```
