> Esta é uma tradução do excelente post feito por [Aditya Bhargava](https://github.com/egonSchiele). O post original pode ser lido  [aqui](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html).
>
> Ao traduzir o post realizei algumas escolhas, tais como:
>1. manter os nomes dos conceitos em inglês, exemplo:  Functors(Funtores) e Monads.
>2. Algumas expressões não fazem sentido se forem traduzidas literalmente para o português, outras perdem o contexto cultural quando você às trás para o Brasil. Por exemplo, Aditya cita Mel Gibson no texto(acredito eu)  pelo uso excessivo de álcool.  Portanto acredito que um substituto para ele seja Zecá Pagodinho, me processe
>
>Se você encontrar qualquer erro na tradução e você encontrará, não utilizei corretores ortográficos, não se acanhe em enviar um pull request.

# Functors, Applicatives e Monads em figuras

A imagem abaixo representa um valor simples

![](img/value.png)

E você sabe como aplicar uma função(ex: adicione +3) cujo o argumento  é o  valor acima,

![](img/value_apply.png)

Muito simples! Uma generalização para o processo acima é dizer que qualquer valor pode estar dentro de um contexto. Um recurso didático é imaginar que o contexto é uma caixa na qual você pode colocar valores dentro

![](img/value_and_context.png)

Agora, quando você aplica uma função a este valor você receberá diferentes resultados **dependendo do  contexto em que o resultado está inserido**. Esta é a ideia que serve de alicerce para  Functors, Applicatives, Monads, Arrows(veja morfismos) etc.  Em se tratando de contextos, o tipo  `Maybe` define dois contextos

![](img/context.png)

```haskell
data Maybe a = Nothing | Just a
```

Posteriormente eu lhe mostrarei as diferenças quando se aplica uma função em algo que é um `Just` ou `Nothing`, mas antes disso vamos conversar um pouco sobre Functors!


## Functors

Quando um valor está envolto por um contexto ele não permite a aplicação de uma função "ordinária" à ele

![](img/no_fmap_ouch.png)



Aqui é onde o `fmap` surge.  O `fmap` tem consciência do contexto. O `fmap` sabe como  atuar  funções em valores
que estão envoltos por um contexto. Por exemplo, suponha que você queira
aplicar `(+3)` em `Just 2`. Usando `fmap` isso é fácil


```haskell
> fmap (+3) (Just 2)
Just 5
```

![](img/fmap_apply.png)

Bum! `fmap` mostra seu valor.  Mas como  `fmap` sabe como  deve aplicar uma função?

### Primieramente, o que é um functor?

Um Functor é chamado  de [typelcass](http://learnyouahaskell.com/types-and-typeclasses#typeclasses-101). A figura a baixo apresenta a definição

![](img/functor_def.png)

Um functor é qualquer tipo de dado que define como `fmap` se aplicará a ele. Aqui, como o `fmap` funciona:

![](img/fmap_def.png)

Então podemos fazer o seguinte

```haskell
> fmap (+3) (Just 2)
Just 5
```

E `fmap` magicamente aplica isto (`Just 2`) a função, pois `Maybe`  é um Functor. A imagem abaixo especifica como `fmap` deve atuar em `Just`'s e `Nothing`'s:

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
*[Bill O’Reilly](https://www.wikiwand.com/en/Bill_O'Reilly_(political_commentator)) sendo  ignorante  sobre `Maybe` (Não conhecia esse cara até traduzir esse post, não confunda com [Tim O'Reilly](https://www.wikiwand.com/pt/Tim_O'Reilly))*

Assim como Morfeu em Matrix  `fmap` sabe o que tem que ser feito. Se você começa com `Nothing` então ele retorna `Nothing`. `fmap` segue o modo zen. Agora faz sentido o porquê da existência do tipo `Maybe`.  Por exemplo, vamos começar a trabalhar com um banco de dados em uma linguagem(nesse caso, ruby) sem `Maybe`:

```ruby
post = Post.find_by_id(1)
if post
  return post.title
else
  return nil
end
```

Mas em Haskell:

```haskell
fmap (getPostTitle) (findPost 1)
```

Se `findPost` retornar um post, o código acima retornará o título do post através de `getPostTitle`. Se `findPost` retornar um `Nothing`, no final ainda teremos `Nothing`! Você precisa concordar que isto é uma forma bem mais organizada  de trabalhar do que utilizar código em ruby.  

No código podemos utilizar  `<$>`, que é uma versão infix de `fmap`, então comumente você poderá encontrar códigos em haskell escritos da seguinte maneira

```haskell
getPostTitle <$> (findPost 1)
```

Aqui em baixo segue um outro exemplo: o que acontece quando você aplica uma função em uma lista?

![](img/fmap_list.png)

Listas são funtores também! Segue uma definição:

```haskell
instance Functor [] where
    fmap = map
```

Okay, okay, um último exemplo: o que acontece quando você aplica uma função em outra função?

```haskell
fmap (+3) (+1)
```

Aqui a função:

![](img/function_with_value.png)

Aqui a função aplicada em outra função:

![](img/fmap_function.png)

O resultado é outra função!

```haskell
> import Control.Applicative
> let foo = fmap (+3) (+2)
> foo 10
15
```

Então funcções são funtores também!

```haskell
instance Functor ((->) r) where
    fmap f g = f . g
```

Quando você usa `fmap` em uma função, você está apenas realizando uma composição de funções!

## Applicatives

Applicatives nos levam ao próximo nível. Com um applicative nossos valores são envoltos por contextos assim como Functors

![](img/value_and_context.png)

Mas é importante notar que no caso de applicatives nossas funções também são envoltas por contextos!

![](img/function_and_context.png)

 Applicatives não estão para brincadeira. `Control.Applicative` define um operador `<*>`, o qual sabe como aplicar uma função envolta por um contexto em um valor envolto por um contexto

![](img/applicative_just.png)

i.e:

```haskell
> Just (+3) <*> Just 2
Just 5
```

O uso de `<*>` pode nos retornar situações deveras interessantes. Por exemplo:

```haskell
> [(*2), (+3)] <*> [1, 2, 3]
[2, 4, 6, 4, 5, 6]
```

![](img/applicative_list.png)

Abaixo segue uma coisa que você consegue fazer com Applicatives, mas não consegue fazer com Functors.

   *Como aplicar uma função que pega dois argumentos para dois valores envoltos em um contexto?*

```haskell
> (+1) <$> (Just 5)
Just (+6)
> Just (+6) <$> (Just 4)
ERRO!!
```



Applicatives:

```haskell
> Just (+6) <*> (Just 3)
Just 8
```

Applicatives colocam Functors  de lado.

"*Adultos podem usar funções com qualquer número de argumentos"-Applicative*


"*Armado com `<$>` and `<*>`, eu posso pegar qualquer função que possui qualquer quantidade de argumentos que não estão envoltos por contextos. Então, eu coloco esses argumentos dentro de contextos, e finalmente eu retorno como saída um valor envolto por contexto huehehuuhe!"-Applicative*

```haskell
> (*) <$> Just 5 <*> Just 3
Just 15
```

Ei! Existe uma função chamada `liftA2` que faz
a mesma coisa:

```haskell
> liftA2 (*) (Just 5) (Just 3)
Just 15
```

## Monads

Como aprender sobre Monads:
1. Obtenha um título de Doutor em Ciência da Computação
2. Jogue ele fora por que você não precisará dele nesta secção!


Monads estão em alta.
Functors aplicam funções a valores dentro de contextos:

![](img/fmap.png)

Applicative aplicam funções que estão dentro de contextos em valores que por sua vez também estão dentro de outros contextos:


![](img/applicative.png)

Monads aplicam funções em valores envoltos por contextos e  **retornam valores envoltos por contextos**. A maquinaria das Monads em haskell é representada pela  função `>>=` (“bind”).
Vejamos um exemplo. Good ol’ `Maybe` is a monad:

![](img/context.png)


Suponha que `half` é uma função que funciona unicamente com números pares

```haskell
half x = if even x
           then Just (x `div` 2)
           else Nothing
```

![](img/half.png)

O que acontece se alimentarmos tal função com um valor envolto por um contexto?

![](img/half_ouch.png)

Nos precisamos utilizar `>>=` em nosso valor com contexto para expulsá-lo do contexto, precisamos retirá-lo da caixa.

 Aqui uma foto representando `>>=`

![](img/plunger.jpg)

Aqui como ele funciona

```haskell
> Just 3 >>= half
Nothing
> Just 4 >>= half
Just 2
> Nothing >>= half
Nothing
```

Mas o que aconteceu dentro da nossa caixa? Monad é um outro `typeclass`. Segue uma definição parcial de uma Monad

```haskell
class Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
Where >>= is:
```

![](img/bind_def.png)

Então uma `Maybe` Monad é
```haskell
instance Monad Maybe where
    Nothing >>= func = Nothing
    Just val >>= func  = func val
```

Aqui a ação desta monad em `Just 3`!

![](img/monad_just.png)

E se você passar um `Nothing` a coisa fica mais simples ainda

![](img/monad_nothing.png)

Você também pode colocar várias dessas ações em cadeia
```haskell
> Just 20 >>= half >>= half >>= half
Nothing
```

![](img/monad_chain.png)

![](img/whoa.png)

Muito legal! Agora vamos nos mexer um pouco e partir para um outro exemplo: a IO monad

![](img/io.png)

Especifiquemos três funções.

1. `getLine` pega os argumentos que são fornecidos por um input do usuário

![](img/getLine.png)

```haskell
getLine :: IO String
```

2. `readFile`  pega uma string(que representa o nome do arquivo) e retorna o conteúdo do arquivo cujo nome é essa string

![](img/readFile.png)
```haskell
readFile :: FilePath -> IO String
```

3. `putStrLn` pega a string e "printa"  ela na tela
![](img/putStrLn.png)

```haskell
putStrLn :: String -> IO ()
```

Todas essas três funções pegam um valor comum(ou nenhum valor) e retornam um valor dentro de um contexto. Nos podemos encadear todas essas ações utilizando `>>=`!

![](img/monad_io.png)

```haskell
getLine >>= readFile >>= putStrLn
```

Aw yeah!  Acomodem-se em suas cadeiras para o show da Monad! Haskell também nos fornece uma sintaxe sucinta para monads, chamada `do notation`:

```haskell
foo = do
    filename <- getLine
    contents <- readFile filename
    putStrLn contents
```

## Conclusão

1. Functor é um tipo que implementa um Functor typeclass.
2. Applicative é um tipo que implementa um Applicative typeclass.
3. Monad  é um tipo que implementa  um Monad typeclass.
4. Maybe implenta todos os outros,  então é um functor, um applicative e um monad.

Qual é a diferença entre os três?
![](img/recap.png)

- **functors:** você aplica uma função em um valor envolto por um contexto utilizando  `fmap` ou `<$>`
- **applicatives:** você aplica uma função envolta por um contexto em um valor envolto por um contexto usando `<*>` ou `liftA`
- **monads:** você  pega um valor envolto por um contexto, extrai ele, aplica uma função, e retorna um valor envolto por um contexto, para isso você pode usar  `>>=` ou `liftM`

Então, querido amigo (Eu penso que neste ponto já posso considerar que somos amigos), eu penso que ambos concordamos que monads são faceis além de serem uma ideia muito inteligente.

Agora que você saboreou esse guia por que não convidar o Zeca Pagodinho para entornar uma cerveja e estudar toda uma secção  sobre monads? Tal secção está disponível [aqui](http://learnyouahaskell.com/a-fistful-of-monads). Existem muitas coisas que eu passei por cima, pois Miran Lipovaca(criador do [Learn You a Haskell](http://learnyouahaskell.com/chapters)) fez um grande trabalho o qual se  aprofundanda em monads e muito outros temas, tal trabalho esta disponível gratuitamente neste link : [http://learnyouahaskell.com/chapters](http://learnyouahaskell.com/chapters)
