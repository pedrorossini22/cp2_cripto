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
