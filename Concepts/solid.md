<h1 align="center">
  <img src="/Images/solid.png" alt="Concepts" width="450px" />
</h1>

# O que é SOLID?

Na *Programação Orientada a Objetos* (POO), **SOLID** é um acrônimo para cinco princípios que facilitam no desenvolvimento de software. Esses princípios podem ser aplicados a qualquer linguagem POO.

Estes princípios foram apresentados por [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) no artigo [The Principles of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)

## Os 5 princípios **S.O.L.I.D.**:

1. **S - Single Responsability Principle** (Príncipio da responsabilidade única)
2. **O - Open-Closed Principle** (Princípio Aberto-Fechado)
3. **L - Liskov Substitution Principle** (Princípio da substituição de Liskov)
4. **I - Interface Segregation Principle** (Princípio da Segregação da Interface)
5. **D - Dependency Inversion Principle** (Princípio da inversão da depêndencia)

Ao seguir estes princípios, o código do desenvolvedor se torna **mais limpo**, mais **modularizado** com pouco **acoplamento**. O que acaba por facilitar a refatoração e reutilização do código.

## SRP -  Príncipio da responsabilidade única
> "Uma classe deve ter apenas uma única responsabilidade"

Esse princípio declara que uma classe deve ser especializada em um **único assunto** e possuir apenas **uma responsabilidade** dentro do software, ou seja, a classe deve ter uma única tarefa ou ação para executar.

Em programação orientada a objetos, uma classe que sabe demais ou faz coisas demais é chamado de **God Class** (Classe Deus). Ao alterar uma **God Class**, sempre existe um nível de incerteza se a alteração de uma responsabilidade
irá comprometer todas as outras, principalmente se não existirem testes automatizados.

### Exemplo prático de uma violação do SRP:

```C#
class Customer
{
    public int id;
    public string name;
	
    public IEnumerable<Customer> GetCustumers() {\*...*\}
    public Order GetCustumerOrders(int customerId) {\*...*\}
    public void AddCustomer(Customer customer) {\*...*\}
    public void DeleteCustomer(Customer customer) {\*...*\}
	
    public void RenderCustomer() {\*...*\}
    public void RenderCustomerOrders() {\*...*\}

    public void Save() {\*...*\}
    public void Load() {\*...*\}
    public void Update() {\*...*\}
    public void Delete() {\*...*\}
	
}
```

A classe *Customer* está violando o SRP pois realiza 3 tipos de tarefas. Ela além de lidar com os dados do *customer*, também é responsável pela renderização e manipulação dos dados.

A violação deste príncipio pode resultar nos seguintes problemas:
* **Falta de coesão** - Uma classe não deve assumir responsabilidades que não são suas
* **Alto acoplamento** - Quanto maior o número de responsabilidades, maior o número de dependências.
* **Dificuldade na implementação de testes automatizados** - É díficil de criar uma *"versão de testes"* dessa classe
* **Dificuldades para reaproveitação do código** - Por ter várias responsabilidades, provavelmente não irá ser reaproveitada em um projeto diferente

### Aplicando SRP na classe *customer*:
```C#
class Customer
{
    public int id;
    public string name;
}

class CustomerService
{
    public IEnumerable<Customer> GetCustumers() {\*...*\}
    public Order GetCustumerOrders(int customerId) {\*...*\}
    public void AddCustomer(Customer customer) {\*...*\}
    public void DeleteCustomer(Customer customer) {\*...*\}
}

class CustomerRenderer
{
    public void RenderCustomer() {\*...*\}
    public void RenderCustomerOrders() {\*...*\}
}

class CustomerRepository
{
    public void Save() {\*...*\}
    public void Load() {\*...*\}
    public void Update() {\*...*\}
    public void Delete() {\*...*\}
}
```
Perceba que conseguimos **quebrar** a classe *customer* em 4 diferentes classes, onde cada uma tem sua própria responsabilidade.

Por reforçar a modularização, o SRP é considerado um dos princípios mais importantes. Ele também é a base para outros princípios e padrões pois lida com questões de coesão e acoplamento, coisa que todo desenvolvedor deve saber aplicar.

## OCP - Príncipio Aberto-Fechado
> "Entidades de software devem ser abertas para extensão, mas fechadas para modificação."

Pessoalmente o meu príncipio favorito, ele prega que quando novos comportamentos e recursos precisarem ser adicionados no software, devemos **estender** e não modificar o código original.

### Exemplo prático de uma violação do OCP:
Suponha que temos um sistema de tarifas por km percorrido para bicicletas e carros. 
```C#
class BikeFare
{
    public double CalculateBikeFare(double distance) { \*...*\ }
}

class CarFare
{
    public double CalculateCarFare(double distance) { \*...*\ }
}

class FareService
{
    public double CalculateFare(double distance, string fareType)
    {
        if (fareType == "bike")
        {
            var fare = new BikeFare();
            return fare.CalculateBikeFare(distance);
        }
        else if (fareType == "car")
        {
            var fare = new CarFare();
            return fare.CalculateCarFare(distance);
        }
    }
}
```
A classe *FareService*, para cara tipo diferente de veiculo, possui uma classe de calculo diferente. Caso surja a necessidade de adicionar suporte a outros veiculos, obviamente teríamos de modificar essa classe, assim violando o princípio Aberto-Fechado do SOLID.
**O principal problema** de alterar uma classe já em funcionamento é a introdução de novos bugs.

Existe uma frase no artigo do **Uncle Bob** que é o seguinte:
> Separate extensible behavioor behind an interface, and flip the dependencies.

Que traduzindo fica:
> Separe o comportamento extensível por trás de uma interface e inverta as dependências

Caso a interface seja bem definida, conseguimos extrair da classe *FareService* a implementação dos tipos de tarifa e chamar somente a **Calculate**.

### Aplicando OCP

Olhando para o nosso exemplo, podemos concluir que o nosso problema é, que para cada novo tipo de veiculo iremos adicionar um novo `if` no programa. Aplicando o OCP e isolando esse comportamento **extensível** atrás de uma interface,
podemos criar uma interface com o nome de **IVehicleFare** contendo o método **Calculate(double distance)**.

Código refatorado:
```C#
interface IVehicleFare
{
    double Calculate(double distance);
}

class BikeFare : IVehicleFare
{
    public double Calculate(double distance) { \*...*\ }
}

class CarFare : IVehicleFare
{
    public double Calculate(double distance) { \*...*\ }
}

class FareService
{
    public double CalculateFare(double distance, IVehicleFare fare)
    {
        return fare.Calculate(distance);
    }
}
```

Com essa alteração, a classe *FareService* não tem mais necessidade de saber qual o tipo de tarifa necessário para chamar o método **Calculate()**. Ela será capaz de calcular a tarifa de qualquer tipo de veículo
que seja criado no futuro (caminhão, avião, etc) sem qualquer necessidade de alteração do código fonte. Com isso implementamos o príncipio Aberto-Fechado em nosso código!


## LSP - Princípio da substituição de Liskov
> "Objetos em um programa devem ser substituíveis por instâncias de seus subtipos, sem alterar a funcionalidade do programa"

Este princípio foi apresentado por [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov), também recebeu o prêmio [Turing Award](https://en.wikipedia.org/wiki/Turing_Award) pelas suas contribuições.
A definição formal de Liskov sobre o príncipio diz que:
> Se para cada objeto o1 do tipo S há um objeto o2 do tipo T de forma que, para todos os programas P definidos em termos de T, 
o comportamento de P é inalterado quando o1 é substituído por o2 então S é um subtipo de T

Para facilitar o entendimento, vamos ao exemplo prático:
```C#
class ClasseBase
{
    public string GetName()
    {
        return "Sou o objeto 'ClasseBase'";
    }
}

class ClasseDerivada : ClasseBase
{
    public new string GetName()
    {
        return "Sou o objeto 'ClasseDerivada'";
    }
}

class Program
{
    public void Main()
    {
        var objeto1 = new ClasseBase();
        var objeto2 = new ClasseDerivada();
        
        Console.WriteLine(objeto1.GetName()); // Sou o objeto 'ClasseBase'
        Console.WriteLine(objeto2.GetName()); // Sou o objeto 'ClasseDerivada'
    }
}
```
Estamos usando a função **GetName** tanto da *ClasseBase* quanto da *ClasseDerivada* e o código funciona da mesma forma para ambas.

### Exemplos prático de violações do LSP:

* Sobrescrever/implementar um método que não faz nada;
* Lançar uma exceção inesperada;
* Retornar valores de tipos diferentes da classe base.

```C#

// Sobrescrevendo um método que não faz nada:
class Penguim : Bird
{
    public override void Fly()
    {
        // Não faz nada
    }
}



// Lançando uma exceção inesperada
class Vehicle
{
    public void Ligar()
    {
        // Liga o veículo
    }
}

class Bike : Vehicle
{
    public new void Ligar()
    {
        throw new Exception("Uma bicicleta não precisa ser ligada");
    }
}


// Retornando valores de tipos diferentes
class Security
{
    public bool Validate()
    {
        // Valida usuário
        return true;
    }
}

class ApiSecurity : Security
{
    public new string Validate()
    {
        //Valida usuário
        return "Usuário válido";
    }
}
```

Ao seguir o **LSP** ganhamos mais confiança para usar polimorfismo por exemplo. Não precisamos nos preocupar com resultados inesperados.

## ISP - Princípio da segregação da interface

## DIP - Princípio da inversão da depêndencia


# Referências
* http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod
* https://medium.com/joaorobertopb/o-que-%C3%A9-solid-o-guia-completo-para-voc%C3%AA-entender-os-5-princ%C3%ADpios-da-poo-2b937b3fc530
* https://pt.wikipedia.org/wiki/SOLID