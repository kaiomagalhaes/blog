A major trend in the startups is the creation of projects in a astronomical speed so that they can be placed in the market to be validated . This being done their destination is defined between a range of options that can be generalized in:**It Needs maintenance or not**. A problem in this approach is the quality of the project in general, mostly on  code  quality and security.

Once the founder of [LinkedIn](www.linkedin.com) Reid Hoffman said:

*If you are not embarrassed by the first version of your product, you’ve launched too late.*

As it applies to what the consumer will find it
should not be applied to the code. Large customizations can not be
necessary, but there are small practices that can ensure good
maintenance as well as saving time ( our most expensive resource ) and
money.

Here at [Codelitt](codelitt.com) most of our projects are built with 
Ruby/Rails. We've defined a set of gems to be used in all projects 
to ensure a minimum of safety and maintainability :

#### Code quality

As quality is something too abstract we've decided to focus in three main points:

  *. Readability
  *. Organization
  *. Maintenability

Our code mus be **readable** in a way that the the developer don't waste too much time
to understand what it does.

Our code must be well **organized** in packages/modules/classes between other things we care about

  *. Class names
  *. Method names
  *. Class groups
  *. Others

Our code must be **maintenable**, although we work with startaps we do care about the future of each one
so if in the future our client decide to improve his product a lot of money and a lot of his time  ( and our time)
will be saved.

##### [Rubocop](https://github.com/bbatsov/rubocop)

RuboCop is a Ruby static code analyzer, it searches for smells and bad practces defined by the community.
Examples of smells are: 

  *. Method too complex
  *. Line number of a class
  *. Characters number in a line
  *. And so on

The official good practces sources are  [Ruby style guide](https://github.com/bbatsov/ruby-style-guide) end
[Rails style guide](https://github.com/bbatsov/rails-style-guide)

It not only have a huge range of good practces plus it allows us to customize them, like in the example below:

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

Besides of showing the code smells it offers a great auto fix that when used with rails 
it does really good updates in the code like turning:

``` ruby
def my_attribute
  my_attribute
end
```

into 

``` ruby
  attr_accessor :my_attribute
```

As soon as I found it i applyed it in a already in production project that I had, it pointed
1400 warnings, after using the autofix the number dropped to 214,  so only a small piece of  it
wasn't automatically fixed and needed the hand of a developer.
 
Along side with [Rubocop](https://github.com/bbatsov/rubocop) we use
[Rubocop Rspec](https://github.com/nevir/rubocop-rspec)  because it has  a lot of RSpec good practices.

Here at [Codelitt](codelitt.com) we use it to garantee good practices in our code,  as sometimes even developers with
years of experience do small mistakes like the wrong use of idention, commas,  variables declared but not used and so on.
We have a politic of 0 warnings in our CI. Before the build we run rubocop and in case of a warning being raised the build fails.

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
