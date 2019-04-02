<p align="center">
<img src="https://www.bill.it/documents/100506/373235/BillBanner.png" alt="Bill Banner">
</p>

[![Version](https://img.shields.io/badge/version-1.0.5-blue.svg)](https://github.com/SisalSpA/bill-consumer-android-sdk/releases)
[![Platform](https://img.shields.io/badge/platform-Android-cc9c00.svg)](https://www.android.com)
[![License](https://img.shields.io/badge/license-Apache%202-lightgrey.svg)](https://github.com/SisalSpA/bill-consumer-android-sdk/blob/master/LICENSE)

---

<p align="center">
    <a href="#features">Features</a> &bull;
    <a href="#requisiti">Requisiti</a> &bull;
    <a href="#installazione-ed-utilizzo">Installazione ed Utilizzo</a> &bull;
    <a href="#ambienti">Ambienti</a> &bull;
    <a href="#documentazione">Documentazione</a> &bull;
    <a href="#changelog">Changelog</a> &bull;
    <a href="#licenza">Licenza</a>
</p>

---

Features
========

Lo strumento di pagamento Bill offre due modalità alternative per completare un pagamento:

-	`App2App`, che tramite un’applicazione esterna consumer permette di inviare all’applicazione Bill una richiesta di pagamento sullo stesso dispositivo
-	`Push2App`, che tramite una notifica push permette di inviare una richiesta di pagamento su dispositivo selezionato dall’utente

Requisiti
=========

| **Versione**  | **Android** |
|---------------|-------------|
| 1.0.0...1.0.4 | 5.0+        |

Installazione ed Utilizzo
=========================

## Gradle
All'interno del file `build.gradle`:
- Aggiungere il repository di Maven Central: `mavenCentral()`
- Aggiungere `implementation "it.sisal.bill:bill-consumer-android-sdk:1.+"`

Ambienti
========

L'SDK prevede il supporto a più ambienti semplicemente aggiungendo alla file della `partnerKey` o del `transactionTokenOnly` il nome dell'ambiente.

Per l'ambiente di produzione non è necessario aggiungere alcun suffisso.

| **Ambiente** | **Suffisso** |
|--------------|--------------|
| Produzione   |              |
| Test         | `-EXTERNAL`  |
> Se per esempio la chiave è `abcdefgh0123456789` questa diventerà `abcdefgh0123456789-EXTERNAL`.

Documentazione
==============

> Tutte le API ("set" esclusa) prevedono l'utilizzo di lambda per la gestione di eventuali risposte.
Queste non verranno chiamate sul main thread, rimane quindi responsabilità dello sviluppatore gestire l'esecuzione del codice all'interno delle lambda sul main thread.

## Inizializzazione

All'avvio dell'app o del processo di pagamento è necessario inizializzare l'SDK utilizzando i parametri `Parnter Key`, `Partner ID` e `Partner Secret` che saranno forniti per ogni integrazione.

| **Parametro**   | **Tipo** |
|-----------------|----------|
| `partnerKey`    | String   |
| `partnerID`     | String   |
| `partnerSecret` | String   |
| `context`       | Context  |

**Kotlin**

```kotlin
BillConsumerSDK.getInstance().set(context, partnerID, partnerKey, partnerSecret)
```

**Java**

```java
BillConsumerSDK.getInstance().set(context, partnerID, partnerKey, partnerSecret);
```

## Richiesta di Pagamento

Il flusso di pagamento prevede che l'applicazione esterna, tramite l'SDK, possa invocare l'applicazione Bill Consumer per effettuare una transazione.

I possibili risultati sono:
-	Impossibilità di proseguire nel pagamento per incompatibilità con l’applicazione Bill Consumer
-	Transazione andata a buon fine
-	Transazione non andata a buon fine con relativo caso di errore da gestire

Il risultato della transazione deve comunque essere verificato tramite la funzionalità di controllo dell'esito della transazione.

### Codice Fiscale e Token di Transazione

Nel caso in cui si sia in possesso di un token di transazione bisogna aggiungere le seguenti chiave-valore nel `Bundle` per l'API `pay`:

#### Richiesta

| **Parametro**       | **Tipo**  | **Obbligatorio** | **Note** |
|---------------------|-----------|------------------|----------|
| `VAT_CODE`          | String    | Si               |          |
| `TRANSACTION_TOKEN` | String    | Si               |          |
| `TIMEOUT`           | Int       | No               | Timeout nel caso in cui l’app Bill non sia installata e venga aperta la WebView per completare il pagamento |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una lambda:

| **Parametro**               | **Note**                                                            |
|-----------------------------|---------------------------------------------------------------------|
| SDK_CONSUMER_GENERIC_ERROR  | Errore generico                                                     |
| SDK_CONSUMER_BILL_OPENED    | L'app Bill è installata ed è stata aperta                           |
| SDK_CONSUMER_TRANSACTION_OK | La WebView è stata aperta e la transazione è andata a buon fine     |
| SDK_CONSUMER_TRANSACTION_KO | La WebView è stata aperta e la transazione non è andata a buon fine |
| SDK_CONSUMER_TIMEOUT        | La WebView è stata aperta ma la transazione è andata in timeout     |

#### Esempio di Codice

**Kotlin**

```kotlin
val parameters = Bundle()
parameters.putString(PayParameters.VAT_CODE.value(), vatCode)
parameters.putString(PayParameters.TRANSACTION_TOKEN.value(), transactionToken)
parameters.putString(PayParameters.TIMEOUT.value(), timeout)
BillConsumerSDK.getInstance().pay(parameters)
```

**Java**

```java
Bundle parameters = new Bundle();
parameters.putString(PayParameters.VAT_CODE.value(), vatCode);
parameters.putString(PayParameters.TRANSACTION_TOKEN.value(), transactionToken);
parameters.putString(PayParameters.TIMEOUT.value(), timeout);
BillConsumerSDK.getInstance().pay(parameters);
```

### Codice Fiscale e Posizione del Merchant

Nel caso in cui si sia in possesso di una posizione di un merchant o quella attuale dell'utente (longitudine e latitudine) bisogna aggiungere le seguenti chiave-valore nel `Bundle` per l'API `pay`:

#### Richiesta

| **Parametro** | **Tipo**        | **Obbligatorio** | **Note** |
|---------------|-----------------|------------------|----------|
| `VAT_CODE`    | String          | Si               |          |
| `AMOUNT`      | Double          | Si               |          |
| `LATITUDE`    | Double          | Si               |          |
| `LONGITUDE`   | Double          | Si               |          |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una lambda:

| **Parametro**                            | **Note**                                                                                |
|------------------------------------------|-----------------------------------------------------------------------------------------|
| SDK_CONSUMER_GENERIC_ERROR               | Errore generico                                                                         |
| SDK_CONSUMER_BILL_INSTALLATION_NOT_VALID | L’app non è presente sul dispositivo oppure il device non rispetta gli standard di Bill |
| SDK_CONSUMER_NO_SHOP                     | Non è stato trovato alcun shop vicino alle coordinate fornite                           |
| SDK_CONSUMER_GENERIC_SHOP_ERROR          | Errore generico con lo shop                                                             |
| SDK_CONSUMER_BILL_OPENED                 | L'app Bill è installata ed è stata aperta                                               |

#### Esempio di Codice

**Kotlin**

```kotlin
val parameters = Bundle()
parameters.putString(PayParameters.VAT_CODE.value(), vatCode)
parameters.putString(PayParameters.AMOUNT.value(), amount)
parameters.putString(PayParameters.LATITUDE.value(), latitude)
parameters.putString(PayParameters.LONGITUDE.value(), longitude)
BillConsumerSDK.getInstance().pay(parameters)
```

**Java**

```java
Bundle parameters = new Bundle();
parameters.putString(PayParameters.VAT_CODE.value(), vatCode);
parameters.putString(PayParameters.AMOUNT.value(), amount);
parameters.putString(PayParameters.LATITUDE.value(), latitude);
parameters.putString(PayParameters.LONGITUDE.value(), longitude);
BillConsumerSDK.getInstance().pay(parameters);
```

### Token di Transazione

Questa API necessita del `Context` per poter funzionare, quindi prima di chimare l'API `pay` è necessario chiamare l'API `set` con solo `Context` come parametro:

**Kotlin**

```kotlin
BillConsumerSDK.getInstance().set(context)
```

**Java**

```java
BillConsumerSDK.getInstance().set(context);
```

Nel caso in cui si sia in possesso del solo token di transazione bisogna aggiungere le seguenti chiave-valore nel `Bundle` per l'API `pay`:

#### Richiesta

| **Parametro**            | **Tipo**        | **Obbligatorio** | **Note** |
|--------------------------|-----------------|------------------|----------|
| `TRANSACTION_TOKEN_ONLY` | String          | Si               | &nbsp;   |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una lambda:

| **Parametro**               | **Note**                                                            |
|-----------------------------|---------------------------------------------------------------------|
| SDK_CONSUMER_GENERIC_ERROR  | Errore generico                                                     |
| SDK_CONSUMER_BILL_OPENED    | L'app Bill è installata ed è stata aperta                           |
| SDK_CONSUMER_TRANSACTION_OK | La WebView è stata aperta e la transazione è andata a buon fine     |
| SDK_CONSUMER_TRANSACTION_KO | La WebView è stata aperta e la transazione non è andata a buon fine |
| SDK_CONSUMER_TIMEOUT        | La WebView è stata aperta ma la transazione è andata in timeout     |

#### Esempio di Codice

**Kotlin**

```kotlin
BillConsumerSDK.getInstance().set(context)

val parameters = Bundle()
parameters.putString(PayParameters.TRANSACTION_TOKEN_ONLY.value(), transactionToken)
BillConsumerSDK.getInstance().pay(parameters)
```

**Java**

```java
BillConsumerSDK.getInstance().set(context);

Bundle parameters = new Bundle();
parameters.putString(PayParameters.TRANSACTION_TOKEN_ONLY.value(), transactionToken);
BillConsumerSDK.getInstance().pay(parameters)
```

## Esito della Transazione

Alla conclusione di un pagamento è necessario controllare l'esito della transazione tramite la funzionalità esposta dall'SDK Bill.

I possibili risultati sono:
-	Transazione andata a buon fine
-	Transazione non andata a buon fine
-	Transazione ancora in corso
-	Transazione in timeout

#### Richiesta

| **Parametro**          | **Tipo** | **Obbligatorio** | **Note** |
|------------------------|----------|------------------|----------|
| `partnerID`            | String   | Si               |          |
| `transactionToken`     | String   | Si               |          |
| `listener`             | Lambda   | Si               | &nbsp;   |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una lambda:

| **Parametro**                     | **Note**                                                            |
|-----------------------------------|---------------------------------------------------------------------|
| SDK_CONSUMER_GENERIC_ERROR        | Errore generico                                                     |
| SDK_CONSUMER_GENERIC_STATUS_ERROR | Errore generico con lo stato della transazione                      |
| SDK_CONSUMER_TRANSACTION_OK       | La WebView è stata aperta e la transazione è andata a buon fine     |
| SDK_CONSUMER_TRANSACTION_KO       | La WebView è stata aperta e la transazione non è andata a buon fine |
| SDK_CONSUMER_TRANSACTION_PENDING  | L'app Bill è installata ed è stata aperta                           |
| SDK_CONSUMER_TRANSACTION_TIMEOUT  | La WebView è stata aperta ma la transazione è andata in timeout     |

#### Esempio di Codice

**Kotlin**

```kotlin
BillConsumerSDK.getInstance().transactionStatus(partnerID: String, transactionToken: String, billResponseListener: BillResponseListener)
```

**Java**

```java
BillConsumerSDK.getInstance().transactionStatus(String partnerID, String transactionToken, BillResponseListener billResponseListener);
```

Changelog
=========

To see what has changed in recent versions of bill-consumer-android-sdk, see the **[CHANGELOG.md](https://github.com/SisalSpA/bill-consumer-android-sdk/blob/master/CHANGELOG.md)** file.

Licenza
=======

bill-consumer-android-sdk is available under the Apache 2.0 license. See the **[LICENSE](https://github.com/SisalSpA/bill-consumer-android-sdk/blob/master/LICENSE)** file for more info.
