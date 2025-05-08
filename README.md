# MercadoBitcoin API Integration Example

Este projeto demonstra como realizar chamadas REST assíncronas utilizando Kotlin, Coroutines e Retrofit em um aplicativo Android. O código faz uma requisição à API do Mercado Bitcoin para buscar informações do ticker (último valor e data) e exibir os resultados na interface do usuário.

## Funcionalidades Principais

- Realiza chamadas assíncronas à API usando Kotlin Coroutines.
- Atualiza a interface do usuário com os dados retornados (valor e data do ticker).
- Realiza o tratamento de erros de respostas HTTP e falhas na chamada.
- Formata valores como moeda brasileira (R$) e datas em um formato amigável.

## Código Explicado

### Função `makeRestCall`

```kotlin
private fun makeRestCall() {
    CoroutineScope(Dispatchers.Main).launch {
        try {
            val service = MercadoBitcoinServiceFactory().create()
            val response = service.getTicker()

            if (response.isSuccessful) {
                val tickerResponse = response.body()

                // Atualizando os componentes TextView
                val lblValue: TextView = findViewById(R.id.lbl_value)
                val lblDate: TextView = findViewById(R.id.lbl_date)

                val lastValue = tickerResponse?.ticker?.last?.toDoubleOrNull()
                if (lastValue != null) {
                    val numberFormat = NumberFormat.getCurrencyInstance(Locale("pt", "BR"))
                    lblValue.text = numberFormat.format(lastValue)
                }

                val date = tickerResponse?.ticker?.date?.let { Date(it * 1000L) }
                val sdf = SimpleDateFormat("dd/MM/yyyy HH:mm:ss", Locale.getDefault())
                lblDate.text = sdf.format(date)

            } else {
                // Trate o erro de resposta não bem-sucedida
                val errorMessage = when (response.code()) {
                    400 -> "Bad Request"
                    401 -> "Unauthorized"
                    403 -> "Forbidden"
                    404 -> "Not Found"
                    else -> "Unknown error"
                }
                Toast.makeText(this@MainActivity, errorMessage, Toast.LENGTH_LONG).show()
            }

        } catch (e: Exception) {
            // Trate o erro de falha na chamada
            Toast.makeText(this@MainActivity, "Falha na chamada: ${e.message}", Toast.LENGTH_LONG).show()
        }
    }
}
```

### Explicação Detalhada

1. **Uso de Coroutines**:
   - A função `makeRestCall` inicia uma coroutine no contexto `Dispatchers.Main`, garantindo que qualquer atualização na interface do usuário seja feita na thread principal.

2. **Serviço Retrofit**:
   - Utiliza um serviço criado pelo `MercadoBitcoinServiceFactory` para realizar a chamada à API. O método `getTicker()` faz a requisição REST.

3. **Tratamento da Resposta**:
   - Se a resposta for bem-sucedida (`response.isSuccessful`):
     - O valor do último preço (`last`) é convertido para `Double` e formatado como moeda brasileira usando `NumberFormat`.
     - A data do ticker (`date`) é convertida de um timestamp Unix para um objeto `Date` e formatada no padrão `dd/MM/yyyy HH:mm:ss`.

4. **Tratamento de Erros HTTP**:
   - Caso a resposta não seja bem-sucedida, exibe uma mensagem ao usuário indicando o erro, com base no código HTTP:
     - `400`: "Bad Request"
     - `401`: "Unauthorized"
     - `403`: "Forbidden"
     - `404`: "Not Found"
     - Outros códigos retornam "Unknown error".

5. **Tratamento de Exceções**:
   - Captura exceções durante a execução e exibe uma mensagem indicando uma falha na chamada.

### Componentes Utilizados

- **`CoroutineScope`**: Gerencia a execução das coroutines.
- **`Dispatchers.Main`**: Garante que a UI seja atualizada na thread principal.
- **`Retrofit`**: Para chamadas REST.
- **`NumberFormat`**: Para formatar valores monetários.
- **`SimpleDateFormat`**: Para formatar datas.

### Dependências Necessárias

Certifique-se de adicionar as seguintes dependências no arquivo `build.gradle`:

```gradle
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4"
implementation "com.squareup.retrofit2:retrofit:2.9.0"
implementation "com.squareup.retrofit2:converter-gson:2.9.0"
```

### Melhorias Futuras

- Adicionar tratamento para outros códigos HTTP.
- Implementar testes unitários e de interface.
- Movimentar a lógica de formatação de dados para uma camada separada.
- Utilizar `ViewModel` e `LiveData` para uma arquitetura mais robusta.

## Como Executar

1. Clone este repositório.
2. Configure o projeto no Android Studio.
3. Certifique-se de que você tenha acesso à API do Mercado Bitcoin.
4. Execute o aplicativo em um dispositivo ou emulador Android.

# MercadoBitcoinServiceFactory

Esta classe é responsável por criar e configurar uma instância do serviço `MercadoBitcoinService` utilizando a biblioteca Retrofit. O Retrofit é uma biblioteca popular em Android para realizar chamadas HTTP de forma simples e eficiente.

## Função Principal

A classe possui um único método, `create()`, que configura e retorna uma instância de `MercadoBitcoinService`. Essa instância é utilizada para realizar chamadas à API do Mercado Bitcoin.

## Código

```kotlin
package carreiras.com.github.cryptomonitor.service

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

class MercadoBitcoinServiceFactory {

    fun create(): MercadoBitcoinService {
        val retrofit = Retrofit.Builder()
            .baseUrl("https://www.mercadobitcoin.net/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()

        return retrofit.create(MercadoBitcoinService::class.java)
    }
}
```

## Explicação do Código

1. **Configuração do Retrofit**:
   - O Retrofit é configurado usando o `Retrofit.Builder()`. Nele são definidos:
     - **`baseUrl`**: A URL base para as chamadas da API. Neste caso, a URL do Mercado Bitcoin (`https://www.mercadobitcoin.net/`).
     - **`addConverterFactory`**: Adiciona um conversor de JSON para objetos Kotlin/Java usando o `GsonConverterFactory`. Isso permite que as respostas da API em formato JSON sejam automaticamente convertidas em objetos do tipo esperado.

2. **Criação do Serviço**:
   - Após a configuração do Retrofit, o método `create()` é chamado para criar uma instância de `MercadoBitcoinService`. Essa interface (que não foi fornecida aqui) define os endpoints e métodos para as chamadas HTTP.

3. **Uso da Classe**:
   - A classe `MercadoBitcoinServiceFactory` é usada para criar uma instância do serviço que será utilizada para realizar chamadas à API. Por exemplo:
     ```kotlin
     val service = MercadoBitcoinServiceFactory().create()
     val response = service.getTicker()
     ```

4. **Facilidade de Manutenção**:
   - Centralizar a lógica de configuração do Retrofit em uma classe separada facilita a manutenção e a reutilização do código. Caso seja necessário alterar a URL base ou o conversor, a modificação é feita em um único local.

## Dependências Necessárias

Certifique-se de adicionar as seguintes dependências no arquivo `build.gradle` para utilizar Retrofit e Gson:

```gradle
implementation "com.squareup.retrofit2:retrofit:2.9.0"
implementation "com.squareup.retrofit2:converter-gson:2.9.0"
```

## Benefícios da Abordagem

- **Modularidade**: A configuração do Retrofit está isolada em uma classe, o que melhora a organização do código.
- **Reutilização**: A mesma lógica pode ser reutilizada em diferentes partes do aplicativo que precisam acessar a API do Mercado Bitcoin.
- **Manutenção Simplificada**: Alterações na URL ou nos conversores são feitas em um único lugar.

## Melhorias Futuras

- Adicionar suporte para diferentes URLs base (por exemplo, ambientes de desenvolvimento e produção).
- Implementar um cliente HTTP customizado para log de requisições ou autenticação.
- Utilizar a biblioteca `OkHttp` para configurar interceptadores de requisições, caso necessário.

## Exemplo de Uso

```kotlin
val service = MercadoBitcoinServiceFactory().create()
val response = service.getTicker()

if (response.isSuccessful) {
    val tickerData = response.body()
    // Processar os dados retornados
} else {
    // Tratar erros
}
```
# MercadoBitcoinService

Esta interface define as operações que o aplicativo pode realizar para interagir com a API do Mercado Bitcoin. Ela utiliza a biblioteca Retrofit para facilitar a comunicação com a API REST.

## Função Principal

A interface `MercadoBitcoinService` contém a definição de um endpoint para buscar informações do ticker (último valor, volume, data, etc.) de Bitcoin.

## Código

```kotlin
package carreiras.com.github.cryptomonitor.service

import carreiras.com.github.cryptomonitor.model.TickerResponse
import retrofit2.Response
import retrofit2.http.GET

interface MercadoBitcoinService {

    @GET("api/BTC/ticker/")
    suspend fun getTicker(): Response<TickerResponse>
}
```

## Explicação do Código

1. **Anotação `@GET`**:
   - Define o tipo de requisição HTTP que será feita. Neste caso, é uma requisição `GET` para o endpoint `api/BTC/ticker/`.
   - O endpoint é relativo à URL base configurada no Retrofit (exemplo: `https://www.mercadobitcoin.net/`).

2. **Método `getTicker`**:
   - Método responsável por fazer a requisição ao endpoint definido.
   - É marcado como `suspend`, o que significa que ele é uma função de suspensão e deve ser usada dentro de uma coroutine ou outra função de suspensão. Isso permite chamadas assíncronas sem bloquear a thread principal.

3. **Retorno do Método**:
   - O método retorna um `Response<TickerResponse>`, onde:
     - **`Response`**: Representa a resposta completa da API, incluindo o código HTTP, cabeçalhos, corpo da resposta, etc.
     - **`TickerResponse`**: Um modelo de dados (não incluído aqui) que mapeia o JSON retornado pela API para um objeto Kotlin.

4. **Uso da Interface**:
   - O Retrofit utiliza esta interface para gerar uma implementação concreta em tempo de execução. Essa implementação pode ser usada para interagir diretamente com a API.
   - Exemplo de uso:
     ```kotlin
     val service = MercadoBitcoinServiceFactory().create()
     val response = service.getTicker()
     if (response.isSuccessful) {
         val tickerData = response.body()
         // Processar os dados do ticker
     } else {
         // Tratar erros de resposta
     }
     ```

## Dependências Necessárias

Certifique-se de incluir as seguintes dependências no arquivo `build.gradle` para usar Retrofit e o suporte a coroutines:

```gradle
implementation "com.squareup.retrofit2:retrofit:2.9.0"
implementation "com.squareup.retrofit2:converter-gson:2.9.0"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4"
```

## Benefícios do Retrofit

- **Simplicidade**: A interface abstrai a complexidade de criar requisições HTTP manualmente.
- **Suporte a Coroutines**: Permite chamadas assíncronas de forma simples e eficiente.
- **Conversão Automática de JSON**: Com o uso de um `ConverterFactory` (como o `GsonConverterFactory`), o Retrofit converte automaticamente os dados JSON da API para objetos Kotlin.

## Melhorias Futuras

- Adicionar tratamento de erros específico para diferentes códigos HTTP.
- Expandir a interface para incluir outros endpoints da API do Mercado Bitcoin.
- Implementar testes para validar as chamadas da API.

## Exemplo de Resposta da API

Abaixo está um exemplo de como a API do Mercado Bitcoin pode retornar dados no endpoint `api/BTC/ticker/`:

```json
{
    "ticker": {
        "high": "300000.00",
        "low": "290000.00",
        "vol": "10.5",
        "last": "295000.00",
        "buy": "294500.00",
        "sell": "295500.00",
        "date": 1683513600
    }
}
```

Esses dados serão mapeados para a classe `TickerResponse` no código.

## Exemplo de Uso em Aplicativo

```kotlin
val service = MercadoBitcoinServiceFactory().create()

CoroutineScope(Dispatchers.IO).launch {
    try {
        val response = service.getTicker()
        if (response.isSuccessful) {
            val tickerData = response.body()
            println("Último preço: ${tickerData?.ticker?.last}")
        } else {
            println("Erro na chamada: ${response.code()}")
        }
    } catch (e: Exception) {
        println("Exceção: ${e.message}")
    }
}
```

Com isso, você consegue acessar os dados do ticker de Bitcoin fornecidos pela API do Mercado Bitcoin.

# TickerResponse e Ticker

Este arquivo define as classes de modelo `TickerResponse` e `Ticker` que representam a estrutura dos dados recebidos da API do Mercado Bitcoin. Essas classes são essenciais para mapear o JSON da resposta da API em objetos Kotlin, facilitando o uso dos dados no aplicativo.

## Estrutura das Classes

### TickerResponse

```kotlin
class TickerResponse(
    val ticker: Ticker
)
```

- **`TickerResponse`**: Esta classe é o modelo principal que representa a resposta completa da API.
- **Propriedades**:
  - `ticker`: Um objeto do tipo `Ticker` que contém os dados relevantes do ticker.

### Ticker

```kotlin
class Ticker(
    val high: String,
    val low: String,
    val vol: String,
    val last: String,
    val buy: String,
    val sell: String,
    val date: Long
)
```

- **`Ticker`**: Esta classe representa os detalhes do ticker retornados pela API.
- **Propriedades**:
  - `high`: O preço mais alto do Bitcoin no período especificado.
  - `low`: O preço mais baixo do Bitcoin no período especificado.
  - `vol`: O volume total negociado no período.
  - `last`: O preço da última negociação realizada.
  - `buy`: O preço de compra mais alto disponível no momento.
  - `sell`: O preço de venda mais baixo disponível no momento.
  - `date`: A data e hora da última atualização do ticker, representada como um timestamp Unix (em segundos).

## Exemplo de Estrutura JSON

A API do Mercado Bitcoin retorna os dados no seguinte formato JSON:

```json
{
    "ticker": {
        "high": "300000.00",
        "low": "290000.00",
        "vol": "10.5",
        "last": "295000.00",
        "buy": "294500.00",
        "sell": "295500.00",
        "date": 1683513600
    }
}
```

As classes `TickerResponse` e `Ticker` mapeiam diretamente essa estrutura, onde:
- O objeto `ticker` é mapeado para a classe `Ticker`.
- Os valores internos de `ticker` (como `high`, `low`, etc.) são atribuídos às propriedades da classe `Ticker`.

## Como Funciona o Mapeamento

Ao usar Retrofit com um conversor como o `GsonConverterFactory`, a biblioteca converte automaticamente o JSON retornado pela API em um objeto do tipo `TickerResponse`. Por exemplo:

```kotlin
val service = MercadoBitcoinServiceFactory().create()
val response = service.getTicker()

if (response.isSuccessful) {
    val tickerResponse = response.body()
    println("Último preço: ${tickerResponse?.ticker?.last}")
}
```

Nesse exemplo:
- O JSON retornado é convertido em um objeto `TickerResponse`.
- O preço da última negociação (`last`) pode ser acessado diretamente como `tickerResponse?.ticker?.last`.

## Benefícios do Modelo

- **Facilidade de Uso**: Permite acessar os dados da API diretamente como propriedades Kotlin.
- **Tipo Seguro**: Usa tipos Kotlin para garantir que os dados recebidos estejam no formato esperado.
- **Integração Simples**: Compatível com Retrofit e Gson para conversão automática de JSON.

## Melhorias Futuras

- Adicionar validação de dados para garantir que os valores recebidos estejam no formato esperado.
- Alterar os tipos de propriedades como `high`, `low`, `vol`, `last`, `buy` e `sell` para `Double` (ao invés de `String`) para facilitar cálculos financeiros.
- Implementar métodos para formatar os valores (por exemplo, formatar o timestamp `date` como uma data legível para humanos).

## Exemplo de Uso

```kotlin
fun printTickerInfo(ticker: Ticker) {
    println("Preço mais alto: ${ticker.high}")
    println("Preço mais baixo: ${ticker.low}")
    println("Volume negociado: ${ticker.vol}")
    println("Último preço: ${ticker.last}")
    println("Preço de compra: ${ticker.buy}")
    println("Preço de venda: ${ticker.sell}")
    println("Data da última atualização: ${Date(ticker.date * 1000L)}")
}
```

Com estas classes, o aplicativo está pronto para processar os dados retornados pela API do Mercado Bitcoin de forma eficiente e organizada.
