Uma grande tendência das statups é a criação de projetos em uma
velocidade astronômica para que os mesmos possam ser colocados no
mercado de forma serem validados. Isso sendo feito o destino dos mesmos
é definido entre uma gama de opções: descontinuado, continuado mas não
precisa de novas funcionalidades, precisa de manutenção etc.


Um problema nesse inicio de projeto é a qualidade do mesmo. Certa vez o
fundador do Likedin Reid Hoffman disse:


*Se você não tem vergonha da primeira versão do seu produto, você
demorou demais para lançar.*


Por mais que se aplique ao produto que o consumidor irá encontrar isso
não deve ser aplicado ao código, grandes customizações não são
necessárias porém existem pequenas práticas que podem garantir uma boa
manutenção no futuro, economia de tempo ( nosso mais caro recurso ) e
dinheiro.


Aqui na [Codelitt](codelitt.com) a maior parte de nossos projetos são
feitos com Ruby/Rails, sendo assim definimos a utilização de um conjunto
minimo de gemas em todos os projetos para garantir um mínimo de
segurança e manutenibilidade:


##### [Rubocop](https://github.com/bbatsov/rubocop)


O Rubocop faz uma análise estática no código buscando por 'ofenças' á
boas práticas definidas pela comunidade, tails como: Complexidade
excessiva em um método, numero de linhas de uma classe, numero de
caracteres em uma linha e etc. Além de nos mostrar os problemas do
código ele nos fornece uma autocorreção simples que pode ser
customizada.


