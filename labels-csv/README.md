## labels-cassava

Import the instances for `FromNamedRecord`:

``` haskell
import Labels.Cassava
```

Then just specify the type you want to load:

``` haskell
> let Right (_,rows :: Vector ("salary" := Int, "name" := Text)) = decodeByName "name,salary\r\nJohn,27\r\n"
> rows
[(#salary := 27,#name := "John")]
```

Non-existent fields or invalid types result in a parse error:

``` haskell
> decodeByName "name,salary\r\nJohn,27\r\n" :: Either String (Header, Vector ("name" := Text, "age" := Int))
Left "parse error (Failed reading: conversion error: Missing field age) at \"\\r\\n\""
> decodeByName "name,salary\r\nJohn,27\r\n" :: Either String (Header, Vector ("name" := Text, "salary" := Char))
Left "parse error (Failed reading: conversion error: expected Char, got \"27\") at \"\\r\\n\""
```

Example with Yahoo!'s market data for AAPL:

``` haskell
> Right (headers,rows :: Vector ("date" := String, "high" := Double, "low" := Double)) <- fmap decodeByName (LB.readFile "AAPL.csv")
> headers
["date","open","high","low","close","volume","adj close"]
```

We can print the rows as-is:

``` haskell
> mapM_ print (V.take 2 rows)
(#date := "2016-08-10",#high := 108.900002,#low := 107.760002)
(#date := "2016-08-09",#high := 108.940002,#low := 108.010002)
```

Accessing fields is natural as anything:

``` haskell
> V.sum (V.map (get #low) rows)
2331.789993
```

We can just make up new fields on the fly:

``` haskell
> let diffed = V.map (\row -> cons (#diff := (get #high row - get #low row)) row) rows
> mapM_ print (V.take 2 diffed)
(#diff := 1.1400000000000006,#date := "2016-08-10",#high := 108.900002,#low := 107.760002)
(#diff := 0.9300000000000068,#date := "2016-08-09",#high := 108.940002,#low := 108.010002)
```

Sometimes a CSV file will have non-valid Haskell identifiers or
spaces, e.g. `adj close` here:

``` haskell
> Right (headers,rows :: Vector ("date" := String, "adj close" := Double)) <- fmap decodeByName (LB.readFile "AAPL.csv")
> mapM_ print (V.take 2 rows)
(#date := "2016-08-10",#adj close := 108.0)
(#date := "2016-08-09",#adj close := 108.809998)
```

Just use the `$("adj close")` syntax:

``` haskell
> mapM_ print (V.take 2 (V.map (get $("adj close")) rows))
108.0
108.809998
```

It still checks the name and type:

``` haskell
> mapM_ print (V.take 2 (V.map (get $("adj closer")) rows))
<interactive>:133:31: error:
    • No instance for (Has
                         "adj closer" a0 ("date" := String, "adj close" := Double))
        arising from a use of ‘get’
````
