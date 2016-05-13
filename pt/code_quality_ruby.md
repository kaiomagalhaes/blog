Uma grande tendência das startups é a criação de projetos em uma
velocidade astronômica para que os mesmos possam ser colocados no
mercado de forma serem validados. Isso sendo feito o destino dos mesmos
é definido entre uma gama de opções: descontinuado, continuado mas não
precisa de novas funcionalidades, precisa de manutenção e etc.

Um problema nesse inicio de projeto é a qualidade do mesmo. Certa vez o
fundador do Linkedin Reid Hoffman disse:

*Se você não tem vergonha da primeira versão do seu produto, você
demorou demais para lançar.*

Por mais que se aplique ao que o consumidor irá encontrar isso
não deve ser aplicado ao código, grandes customizações podem não ser
necessárias, porém existem pequenas práticas que podem garantir uma boa
manutenção no futuro além, de  economia de tempo ( nosso mais caro recurso ) e
dinheiro.

Aqui na [Codelitt](codelitt.com) a maior parte dos nossos projetos são
feitos com Ruby/Rails, sendo assim definimos a utilização de um conjunto
 de gemas a serem utilizadas em todos os projetos para garantir um mínimo de
segurança e manutenibilidade:

#### Qualidade de código

Sobre qualidade de código nós focamos em três pontos chave: Leitura,
organização e manutenção.

Nosso código tem que ser **legível** de forma que o desenvolvedor não gaste
muito tempo para entender o que está sendo feito.

Nosso código tem que ser **organizado** de forma fazer sentido
 arquiteturalmente, ex:

  1. Nomeclatura das classes
  2. Nomeclatura dos métodos
  3. Classes com objetivo semelhantes agrupadas em módulos
  4. Entre outros

Nosso código tem que ser **manutenível**. Por mais que trabalhemos com
startups nós nos preocupamos com a continuidade dos projetos de forma
que seja possivel economizar tempo caso alterações sejam necessárias no
futuro.

##### [Rubocop](https://github.com/bbatsov/rubocop)

O Rubocop faz uma análise estática no código buscando por 'ofensas' 
(onde cada uma gera uma warning) á boas práticas definidas pela comunidade,
tais como: Complexidade excessiva em um método, numero de linhas de uma 
classe, numero de caracteres em uma linha e etc.

As fontes oficiais de boas práticas são: [Ruby style
guide](https://github.com/bbatsov/ruby-style-guide) e [Rails style
guide](https://github.com/bbatsov/rails-style-guide)

Ele não só possui uma gama imensa de boas práticas pré definidas como
também nos permite a customização das mesmas, como por exemplo: 

```
Rails:
  Enabled: true

Metrics/LineLength:
  Max: 120

Style/Documentation:
  Enabled: false

Style/BlockDelimiters:
  Enabled: false
```

Além de nos mostrar os problemas do código ele nos fornece um ótimo
sistema de autocorreção que quando utilizado com rails faz belas
alterações no projeto, sempre pensando em boas práticas. Assim que
descobri essa gema decidi aplica-la em um projeto em desenvolvimento,
inicialmente ele apontou 1400 warnings, após a utilização da
autocorreção  esse número pulou para 214, ou seja, do valor inicial
somente uma pequena parcela não foi corrigida automaticamente.

Juntamente [Rubocop](https://github.com/bbatsov/rubocop) utilizamos o
[Rubocop Rspec](https://github.com/nevir/rubocop-rspec) sempre que
utilizamos o [RSpec](https://github.com/rspec/rspec) pois ele possui
um foco maior nos testes.

Na [Codelitt](codelitt.com) nós o utilizamos para garantir a aplicação
das boas práticas, por mais que tenhamos bons desenvolvedores muitas
vezes não percebemos pequenos detalhes como espaçamento, tamanho de linha,
a não utilização de uma variavel e etc. Temos uma política de 0 warnings
no nosso CI. Antes de fazer a build o rubocop é ativado, e caso alguma ofensa
seja encontrada a mesma falha imediatamente.

##### [Rubycritic](https://github.com/whitesmith/rubycritic)

O Rubycritic é outra ferramenta de análise estática de código. Sendo
feita a partir da junção de outras três gemas
 ([Reek](https://github.com/troessner/reek),
 [Flay](https://github.com/seattlerb/flay) e
 [Flog](https://github.com/seattlerb/flog)), ela nos
fornece uma visão geral do projeto e uma visão por classes.

O ponto alto dessa gema são as análises de codigo duplicado e
complexidade, como pode ser visto na imagem do relatório abaixo:

![alt text](http://www.clipular.com/c/5227312822353920.png?k=xKPmaAjaIBnIg-ZwOJoLbZVlQZ8
"Ruby Critics image example")

Aqui [Codelitt](codelitt.com) nos preocupamos bastante com o principio
[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) e
as análises de code smells e código duplicado nos auxiliam 
em sua aplicação.

#### Segurança

Como trabalhamos com projetos que vão desde de simples sites como sistemas
que envolvem transações financeiras, mantemos por padrão uma grande
preocupação com segurança.

Além de práticas em servidores e cuidados com dados sensiveis, obviamente
não podemos deixar de nos preocupar com ataques feitos ás nossas aplicações
como por exemplo: DDOS, SQL Injection e etc. Para garantir que
nossas aplicações estejam minimamente seguras nós utilizamos as seguintes
gemas:

###### [Brakeman](https://github.com/presidentbeef/brakeman)

O Brakeman faz uma analise estática no código buscando falhas de segurança
no mesmo.

Aqui na [Codelitt](codelitt.com) nossa politica é de 0 warnings. Sendo
esse é o nosso primeiro passo na build, caso alguma brecha de segurança
seja encontrada o projeto nem mesmo é construido.

###### [Dawnscanner](https://github.com/thesp0nge/dawnscanner)

O Dawnscanner é outra gema que faz analise estática no código em busca
de falhas na segurança.
Utilizamos ele juntamente com [Brakeman](https://github.com/presidentbeef/brakeman)
para garantir uma redundância de verificação.
Somente após a verificação com o [Brakeman](https://github.com/presidentbeef/brakeman) 
nós verificamos com o [Dawnscanner](https://github.com/thesp0nge/dawnscanner)
e caso ele encontre a build do projeto irá falhar.

###### [Bundler-audit](https://github.com/rubysec/bundler-audit)

O bundler audit é uma gema que verifica o Gemfile.lock e busca por falhas
reportadas na versão das gemas utilizadas, caso encontre ele nos indica
uma versão mais segura da mesma caso os problemas tenham sido corrigidos
e caso não exista, bom, é hora de buscar por uma outra. 

Durante o desenvolvimento de uma aplicação é comum que nos importemos 
com falhas de segurança no projeto, no entanto caso uma gema utilizada esteja 
comprometida todo o projeto também estará.

Aqui na [Codelitt](codelitt.com) a verificação com o 
[Bundler-audit](https://github.com/rubysec/bundler-audit) é o terceiro passo
antes da build. Quando uma gema está comprometida nós buscamos por uma versão
que não esteja e caso não a encontremos tratamos de buscar uma outra solução e
a removemos de nossas aplicações.

#### Cobertura de código

Cobertura de código é algo importante pois a partir desse dado nós podemos
garantir que a aplicação está sendo minimamente testada. Vale ressaltar que
isso não garante que os pontos críticos da aplicação estão sendo avaliados,
o que é o nosso foco aqui: Garantir que tudo o que pode dar algum tipo de 
problema seja testado para que fique claro quando alguma implementação o 
quebrar.

##### [Simplecov](https://github.com/colszowka/simplecov)

O SimpleCov é uma gema que faz uma análise dinâmica da cobertura de
código, costumamos utiliza-la em conjunto com o RSpec. Além de oferecer
relatórios sobre os testes ela também permite que os mesmos falhem caso
uma porcentagem mínima não seja atingida.

Aqui na [Codelitt](codelitt.com) definimos que o mínimo de cobertura
deverá ser 90%, assim garantimos sempre que nossos projetos estejam
sendo sendo bem testados.
