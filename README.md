# Functors, Applicatives e Monads em figuras

A imagem abaixo representa um valor simples

![](img/value.png)

E você sabe como aplicar uma função(ex: adicione +3) cujo o argumento  é o  valor acima,

![](img/value_apply.png)

Muito simples! Uma generalização para o processo acima é dizer que qualquer valor pode estar dentro de um contexto. Um recurso didático é imaginar que o contexto é uma caixa na qual você pode colocar valores dentro

![](img/value_and_context.png)

Agora, quando você aplica uma função a este valor você receberá diferentes resultados dependendo do seu contexto. Esta é a ideia que serve de alicerce para  Functors, Applicatives, Monads, Arrows(veja morfismos) etc.  Em se tratando de contextos, o tipo  `Maybe` define dois contextos

Posteriormente eu lhe mostrarei as diferenças quando se aplica uma função em algo que é um `Just` ou `Nothing`, mas antes disso vamos conversar um pouco sobre Functors!


## Functors

Quando um valor está envolto por um contexto ele não permite a aplicação de uma função "ordinária" à ele

![](img/no_fmap_ouch.png)



Aqui é onde o `fmap` surge.  O `fmap` tem consciência do contexto. O `fmap` sabe como  atuar  funções em valores
que estão envoltos por um contexto. Por exemplo, suponha que você queira
aplicar `(+3)` para `Just 2`. Usando `fmap` isso é facil


```haskell
> fmap (+3) (Just 2)
Just 5
```

![](img/fmap_apply.png)

Bum! `fmap` mostra seu valor.  Mas como  `fmap` sabe como  deve aplicar uma função?

### Primieramente, o que é um functor?

Um Functor é chamado  de typelcass. A figura a baixo apresenta a definição

![](img/functor_def.png)

Um functor é qualquer tipo de dado que define como `fmap` se aplicará a ele. Aqui como o `fmap` funciona

![](img/fmap_def.png)

Então podemos fazer o seguinte

```haskell
> fmap (+3) (Just 2)
Just 5
```

E `fmap` magicamente aplica isto a função, pois `Maybe`  é um Functor. A imagem abaixo especifica como `fmap` deve atuar em `Just`'s e `Nothing`'s:

```haskell
instance Functor Maybe where
    fmap func (Just val) = Just (func val)
    fmap func Nothing = Nothing
```



O que acontece quando escrevemos `fmap +(3) (Just 2)`

![](img/fmap_just.png)

Você gostou? Certo `fmap`, então aplique `(+3)` em um  `Nothing`

![](img/fmap_nothing.png)


```haskell
> fmap (+3) Nothing
Nothing
```

![](img/bill.png)

Bill O’Reilly sendo um ignorante total sobre `Maybe` totally ignorant about the Maybe functor

Like Morpheus in the Matrix, `fmap` knows just what to do; you start with Nothing, and you end up with Nothing! `fmap` is zen. Now it makes sense why the `Maybe` data type exists. For example, here’s how you work with a database record in a language without `Maybe`:

```
post = Post.find_by_id(1)
if post
  return post.title
else
  return nil
end
```

But in Haskell:

```haskell
fmap (getPostTitle) (findPost 1)
```

If `findPost` returns a post, we will get the title with `getPostTitle`. If it returns `Nothing`, we will return `Nothing`! Pretty neat, huh? `<$>` is the infix version of `fmap`, so you will often see this instead:

```
getPostTitle <$> (findPost 1)
```

Here’s another example: what happens when you apply a function to a list?

![](img/fmap_nothing.png)

Lists are functors too! Here’s the definition:

```haskell
instance Functor [] where
    fmap = map
```

Okay, okay, one last example: what happens when you apply a function to another function?

```haskell
fmap (+3) (+1)
```

Here’s a function:

![](img/function_with_value.png)

Here’s a function applied to another function:

![](img/fmap_function.png)

The result is just another function!

```haskell
> import Control.Applicative
> let foo = fmap (+3) (+2)
> foo 10
15
```

So functions are Functors too!

```haskell
instance Functor ((->) r) where
    fmap f g = f . g
```

When you use fmap on a function, you’re just doing function composition!

## Applicatives

Applicatives take it to the next level. With an applicative, our values are wrapped in a context, just like Functors:

![](img/value_and_context.png)

But our functions are wrapped in a context too!

![](img/function_and_context.png)

Yeah. Let that sink in. Applicatives don’t kid around. Control.Applicative defines `<*>`, which knows how to apply a function wrapped in a context to a value wrapped in a context:

![](img/applicative_just.png)

i.e:
```haskell
Just (+3) <*> Just 2 == Just 5
```

Using `<*>` can lead to some interesting situations. For example:

```haskell
> [(*2), (+3)] <*> [1, 2, 3]
[2, 4, 6, 4, 5, 6]
```

![](img/applicative_list.png)

Here’s something you can do with Applicatives that you can’t do with Functors. How do you apply a function that takes two arguments to two wrapped values?

```haskell
> (+) <$> (Just 5)
Just (+5)
> Just (+5) <$> (Just 4)
ERROR ??? WHAT DOES THIS EVEN MEAN WHY IS THE FUNCTION WRAPPED IN A JUST
```

Applicatives:

```haskell
> (+) <$> (Just 5)
Just (+5)
> Just (+5) <*> (Just 3)
Just 8
```

Applicative pushes Functor aside. “Big boys can use functions with any number of arguments,” it says. “Armed <$> and `<*>`,
 I can take any function that expects any number of unwrapped values. Then I pass it all wrapped values, and I get a wrapped value out! AHAHAHAHAH!”

```
> (*) <$> Just 5 <*> Just 3
Just 15
And hey! There’s a function called liftA2 that does the same thing:
> liftA2 (*) (Just 5) (Just 3)
Just 15
```

## Monads

Como aprender sobre Monads:
1. Obtenha um titulo de Doutor em Ciência da Computação
2. Jogue ele fora por que você não precisará dele nesta secção!


Monads add a new twist.
Functors aplicam funções a valores dentro de contextos:

![](img/fmap.png)

Applicative aplicam funções que estão dentro de contextos em valores que por sua vez também estão dentro de outros contextos:


![](img/applicative.png)

Monads aplicam funções em valores envoltos por contextos e  retornam valores envoltos por contextos. A maquinaria das Monads em haskell é representada pela  função >>= (“bind”).
Vejamos um exemplo. Good ol’ Maybe is a monad:

![](img/context.png)

Just a monad hanging out
Suponha que `half` é uma função que funciona unicamente em números pares

```
half x = if even x
           then Just (x `div` 2)
           else Nothing
```

![](img/half.png)

O que acontece se alimentarmos tal função com um valor envolto por um contexto?

![](img/half_ouch.png)

Nos precisamos utilizar `>>=` em nosso valor com contexto para expulsá-lo do contexto, retirá-lo da caixa. Aqui uma foto representando `>>=`

![](img/plunger.png)

Aqui como ele funciona
```
> Just 3 >>= half
Nothing
> Just 4 >>= half
Just 2
> Nothing >>= half
Nothing
```

Mas o que aconteceu dentro da nossa caixa? Monad é um outro `typeclass`. Segue uma definição parcial de uma Monad
```
class Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
Where >>= is:
```
![](img/bind_def.png)

Então uma `Maybe` Monad é
```

instance Monad Maybe where
    Nothing >>= func = Nothing
    Just val >>= func  = func val
```

Aqui a ação desta monad em `Just 3`!
![](img/monad_just.png)

E se você passar um `Nothing` a coisa fica mais simples ainda
![](img/monad_nothing.png)

Você também pode colocar várias dessas ações em cadeia
```
> Just 20 >>= half >>= half >>= half
Nothing
```

![](img/monad_chain.png)
![](img/whoa.png)

Muito legal! Agora vamos nos mexer um pouco e partir para um outro exemplo: a IO monad

![](img/io.png)

Especificamente três funções. `getLine` pega os argumentos que são fornecidos por um input do usuário
![](img/getLine.png)

```haskell
getLine :: IO String
```

`readFile`  pega uma string(que representa o nome do arquivo) e retorna o conteudo do arquivo cujo nome é essa string
![](img/readFile.png)
```haskell
readFile :: FilePath -> IO String
```

`putStrLn` pega a string e "printa"  ela na tela
![](img/putStrLn.png)

```haskell
putStrLn :: String -> IO ()
```

Todas essas três funções pegam um valor comum(ou nenhum valor) e retornam um valor dentro de um contexto. Nos podemos encadear todas essas ações utilizando `>>=`!

![](img/monad_io.png)

```haskell
getLine >>= readFile >>= putStrLn
```

Aw yeah!  Acomodem-se em suas cadeiras para o show da Monad! Haskell também nos fornece um sintaxe sucinta para monads, chamada do notation:

```haskell
foo = do
    filename <- getLine
    contents <- readFile filename
    putStrLn contents
```

## Conclusion

A functor is a data type that implements the Functor typeclass.
An applicative is a data type that implements the Applicative typeclass.
A monad is a data type that implements the Monad typeclass.
A Maybe implements all three, so it is a functor, an applicative, and a monad.
What is the difference between the three?
![](img/recap.png)
