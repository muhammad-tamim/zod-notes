<h1 align="center">Zod Notes</h1>

- [Zod:](#zod)
  - [Basic:](#basic)
    - [Defining a schema:](#defining-a-schema)
    - [Parsing data:](#parsing-data)
    - [Handling errors:](#handling-errors)
    - [Inferring types:](#inferring-types)
  - [Strings Validations:](#strings-validations)
    - [Number Validations:](#number-validations)
  - [Type Coercion:](#type-coercion)


# Zod: 

Zod is a TypeScript-first schema validation library. We use Zod to validate incoming request body, params, and queries before saving them to the database.

## Basic:

### Defining a schema:
A schema is simply a blueprint that describes what valid data should look like.

```ts
import * as z from "zod"; 
 
const Player = z.object({ 
  username: z.string(),
  xp: z.number()
});
```

### Parsing data:
use `.parse` to validate an schema input. If it's valid, Zod returns a strongly-typed deep clone of the input.

```ts
const reqBody = {userName: "Tamim", xp: 100} 

Player.parse(reqBody); 
// => returns { username: "billie", xp: 100 }
```

### Handling errors:
When validation fails, the .parse() method will throw a ZodError instance with granular information about the validation issues.

```ts
const reqBody = {userName: 42, xp: "100"} 

try {
  Player.parse(reqBody);
} catch(error){
  if(error instanceof z.ZodError){
    error.issues; 
    /* [
      {
        expected: 'string',
        code: 'invalid_type',
        path: [ 'username' ],
        message: 'Invalid input: expected string'
      },
      {
        expected: 'number',
        code: 'invalid_type',
        path: [ 'xp' ],
        message: 'Invalid input: expected number'
      }
    ] */
  }
}
```

To avoid a try/catch block, you can use the .safeParse() method to get back a plain result object containing either the successfully parsed data or a ZodError.

```ts
const reqBody = { username: 42, xp: "100" }

const result = Player.safeParse(reqBody);
if (!result.success) {
  result.error;   // ZodError instance
} else {
  result.data;    // { username: string; xp: number }
}
```

### Inferring types: 
Zod infers a static type from your schema definitions. You can extract this type with the z.infer<> utility and use it however you like.

```ts
const Player = z.object({ 
  username: z.string(),
  xp: z.number()
});
 
// extract the inferred type
type PlayerType = z.infer<typeof Player>;
 
// use it in your code
const reqBody: PlayerType = { username: "billie", xp: 100 };
```

## Strings Validations: 
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

To validate against some common string formats:

```ts
z.email();         // /^(?!\.)(?!.*\.\.)([a-z0-9_'+\-\.]*)[a-z0-9_+-]@([a-z0-9][a-z0-9\-]*\.)+[a-z]{2,}$/i
z.email({ pattern: /your custom regex here/ });

z.url();
z.url({ hostname: /^example\.com$/ });
z.url({ protocol: /^https$/ });
z.url({ protocol: /^https?$/, hostname: z.regexes.domain}); 
// z.regexes.domain = /^([a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\.)+[a-zA-Z]{2,}$/
z.httpUrl();       // http or https URLs only

z.hex();
z.jwt();
z.jwt({ alg: "HS256" });
z.iso.date();
z.iso.time();
z.iso.datetime();
z.iso.duration();
```

### Number Validations: 

```ts
z.number()
z.number().gt(5);
z.number().gte(5);                     // alias .min(5)
z.number().lt(5);
z.number().lte(5);                     // alias .max(5)
z.number().positive();       
z.number().nonnegative();    
z.number().negative(); 
z.number().nonpositive(); 
z.number().multipleOf(5);              // alias .step(5)
```

## Type Coercion: 
For type coercion input data to the appropriate type, use z.coerce instead:

```ts
z.coercion.string();    // String(input)
z.coercion.number();    // Number(input)
z.coercion.boolean();   // Boolean(input)
z.coercion.bigint();    // BigInt(input)
```

```ts
const schema = z.coerce.string();
 
schema.parse("tuna");    // => "tuna"
schema.parse(42);        // => "42"
schema.parse(true);      // => "true"
schema.parse(null);      // => "null"
```

The input type of these coerced schemas is unknown by default. To specify a more specific input type, pass a generic parameter:

```ts
const A = z.coerce.number();
type AInput = z.input<typeof A>; // => unknown
 
const B = z.coerce.number<number>();
type BInput = z.input<typeof B>; // => number
```
