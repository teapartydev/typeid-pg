# TypeId-Pg

## An Implementation of TypeId for Postgres

TypeId is a type-safe, globally unique identifier format inspired by Stripe's IDs, designed to be sortable, unique and type-safe.

This implementation of TypeId does some modification to the base TypeId specification by replacing the random UUID part of the TypeId in favor of a sortable ULID

This implement of TypeId consists of two parts a `Type` prefix and a `ULID` suffix, separated by an underscore.

- ```pseudo
  user_01HXYG96MV7WS0E2KW4ZCGP3KZ
  └──┘ └────────────────────────┘
  Type          ULID
  ```

- **Type Prefix**
  - The type prefix is a string of up to 63 characters in lowercase snake_case ASCII (a-z and underscore). It indicates the type of entity the TypeId represents (e.g., user, post, etc.). This helps prevent accidental misuse of IDs and aids in understanding the entity type.
- **ULID Suffix**
  - Following the underscore is a 26-character string that represents a ULID. This part ensures the TypeId's global uniqueness and this also makes TypeId's sortable.

This implementations TypeId offers a modern and efficient way to handle globally unique identifiers in a type-safe manner, with enhanced sorting capabilities.

## Usage of Pg-TypeId

Once you've installed the TypeID types and functions from `typeid.sql` in your Postgres instance, you can use it as follows.

- Generating TypeId

  - ```sql
    -- Generate a TypeId
    select gen_typeid('user') 
    -- Result: user_01HXZF9VT3ANNTRE39FBXDGHBM 
    ```

- Validating TypeId

  - ```sql
    -- If TypeId is valid
    select check_typeid('user_01HXZF9VT3ANNTRE39FBXDGHBM', 'user') 
    -- Result: true

    -- If TypeId is invalid
    select check_typeid('01HXZF9VT3ANNTRE39FBXDGHBM', 'user')
    -- Result: false
    ```

- Defining TypeId

  - ```sql
    -- Defining a users table with TypeId of type `user` as primary key
    create table users (
      -- The id is a TypeId of with the type of `user`.   
      -- The id also has a default constraint of `gen_typeid` to autogenerated TypeId with with the correct type of `user`. 
      -- The id also has a check constraint of `check_typeid` to check if it's a valid TypeId and has the correct type of `user`.
      id varchar(256) default gen_typeid('user') not null check(check_typeid(id, 'user')) primary key,
      name varchar(256) not null,
      email varchar(256) not null,
      created_at timestamptz default now() not null,
      updated_at timestamptz
    );
    ```
