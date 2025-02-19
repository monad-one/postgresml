# Vector Operations

PostgresML adds optimized vector operations that can be used inside SQL queries. Vector operations are particularly useful for dealing with embeddings that have been generated from other machine learning algorithms, and can provide functions like nearest neighbor calculations using various distance functions.

Embeddings can be a relatively efficient mechanism to leverage the power of deep learning, without the runtime inference costs. These functions are fast with the most expensive distance functions computing upwards of ~100k per second for a memory resident dataset on modern hardware.

The PostgreSQL planner will also [automatically parallelize](https://www.postgresql.org/docs/current/parallel-query.html) evaluation on larger datasets, if configured to take advantage of multiple CPU cores when available.

Vector operations are implemented in Rust using `ndarray` and BLAS, for maximum performance.

## Element-wise Arithmetic with Constants

#### Addition

```postgresql
pgml.add(a REAL[], b REAL) -> REAL[]
```

=== "SQL"

    ```postgresql
    SELECT pgml.add(ARRAY[1.0, 2.0, 3.0], 3);
    ```

=== "Ouput"

    ```
    pgml=# SELECT pgml.add(ARRAY[1.0, 2.0, 3.0], 3);
       add
    ---------
     {4,5,6}
    (1 row)
    ```

#### Subtraction
```postgresql
pgml.subtract(minuend REAL[], subtrahend REAL) -> REAL[]
```

#### Multiplication
```postgresql
pgml.multiply(multiplicand REAL[], multiplier REAL) -> REAL[]
```

#### Division
```postgresql
pgml.divide(dividend REAL[], divisor REAL) -> REAL[]
```

## Pairwise arithmetic with Vectors

#### Addition
```postgresql
pgml.add(a REAL[], b REAL[]) -> REAL[]
```

#### Subtraction
```postgresql
pgml.subtract(minuend REAL[], subtrahend REAL[]) -> REAL[]
```

#### Multiplication
```postgresql
pgml.multiply(multiplicand REAL[], multiplier REAL[]) -> REAL[]
```

#### Division
```postgresql
pgml.divide(dividend REAL[], divisor REAL[]) -> REAL[]
```

## Norms

#### Dimensions not at origin
```postgresql
pgml.norm_l0(vector REAL[]) -> REAL
```

#### Manhattan distance from origin
```postgresql
pgml.norm_l1(vector REAL[]) -> REAL 
```

#### Euclidean distance from origin
```postgresql
pgml.norm_l2(vector REAL[]) -> REAL 
```

#### Absolute value of largest element
```postgresql
pgml.norm_max(vector REAL[]) -> REAL 
```

## Normalization

#### Unit Vector
```postgresql
pgml.normalize_l1(vector REAL[]) -> REAL[]
```

#### Squared Unit Vector
```postgresql
pgml.normalize_l2(vector REAL[]) -> REAL[]
```

#### -1:1 values
```postgresql
pgml.normalize_max(vector REAL[]) -> REAL[]
```

## Distances

#### Manhattan
```postgresql
pgml.distance_l1(a REAL[], b REAL[]) -> REAL
```

#### Euclidean
```postgresql
pgml.distance_l2(a REAL[], b REAL[]) -> REAL
```

#### Projection
```postgresql
pgml.dot_product(a REAL[], b REAL[]) -> REAL
```

#### Direction
```postgresql
pgml.cosine_similarity(a REAL[], b REAL[]) -> REAL
```

## Nearest Neighbor Example

If we had precalculated the embeddings for a set of user and product data, we could find the 100 best products for a user with a similarity search.

```postgresql
SELECT 
    products.id, 
    pgml.cosine_similarity(
        users.embedding,
        products.embedding
    ) AS distance
FROM users
JOIN products
WHERE users.id = 123
ORDER BY distance ASC
LIMIT 100;
```
