# PSD2 API Guide
- Dynamic sandbox environment that fully meets the PSD2 requirements for providing APIs for Third-Party Providers (TPP). Based on the Berlin Group’s NextGen PSD2 specification for access to accounts (XS2A).
- The basic system is based on adorsys xs2a-sandbox. (https://github.com/adorsys/XS2A-Sandbox)
- The entire API list for each module can be checked through each swagger file.
>- Tpp Rest Server : Host:8093/swagger-ui.html
>- Banking Rest Server : Host:8090/swagger-ui.html
>- Core rest Server : Host:8089/swagger-ui.html

## Use Cases
- 1. Login
- 2. Create User
- 3. Depoist Cash
- 4. Transfer
- 5. Get Account Balance & Transactions

## 1. Login
- TPP login & Banking login
>- TPP login : Third-Party Providers admin login, It's possible to register new user after login.
>- Banking login : User online banking login, It's possible to get account information(balance, transaction etc..) after login.

### 1.1 TPP login
- path : {{host}}:8093/tpp/login
- POST
- Request
>- Header
>>- login : TPP id
>>- pin : TPP password

- Response (success)
>- Http Status : 200 OK
>- Header
>>- access_token : It's required to use other TPP apis.

### 1.2 Banking login
- path : {{host}}:8090/api/v1/login
- POST
- Request
>- Header
>>- login : PSU ID
>>- pin : banking user password

- Response (success)
>- Http Status : 200 OK
>- Header
>>- access_token : It's required to use other banking apis.

## 2. Create User
- This case consists of a total of three processes. Creating new users, issuing new ibans, and mapping between users and ibans

### 2.1 Create new user
- path : {{host}}:8093/tpp/users
- POST
- Request
>- Header
>>- Authorization : access_token from response of TPP login
>- body (example)
```
{
    "accountAccesses": [],
    "blocked": false,
    "branch": {{Tpp branch}}, <== it's tpp admin branch 
    "email": "tester1@mail.com",
    "login": "tester1",
    "pin": "12345",
    "scaUserData": [],
    "systemBlocked": false,
    "userRoles": ["CUSTOMER"]
}
```

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
{
    "id": "xwrCtgDBQRwi7P6OUj1FJk",
    "login": "tester1",
    "email": "tester1@mail.com",
    "scaUserData": [],
    "accountAccesses": [],
    "userRoles": [
        "CUSTOMER"
    ],
    "branch": tpp branch,
    "blocked": false,
    "systemBlocked": false
}
```

### 2.2 Issuing new ibans
- path : {{host}}:8093/tpp/data/generate/iban?tppId={{Tpp branch}}
- GET
- Request
>- Header
>>- Authorization : access_token from response of TPP login
>- Param
>>- tppId : tpp branch ID

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
GB23YKGU10834700000115
```

### 2.3 Mapping between users and ibans
- path : {{host}}:8093/tpp/accounts?userId={{user id}}
- POST
- Request
>- Header
>>- Authorization : access_token from response of TPP login
>- param
>>- userId : ID issued when a new user is created (response of Create new user)
>- body (example)
```
{
    "accountStatus": "ENABLED",
    "accountType": "CASH",
    "currency": "EUR",
    "iban": {{issued iban}},
    "id": {{user id}},
    "usageType": "PRIV"
}
```

- Response (success)
>- Http Status : 200 OK

## 3. Depoist Cash
- The process of depositing money with the issued iban
- First, you need to obtain account access information.

### 3.1 Get account access information
- path : {{host}}:8093/tpp/users/{{user id}}
- GET
- Request
>- Header
>>- Authorization : access_token from response of TPP login
>- path param
>>- {{user id}} : ID issued when a new user is created (response of Create new user)

- Response (success)
>- Http Status : 200 OK
>- body (example)
```
{
    "id": user id,
    "login": "tester1",
    "email": "tester1@mail.com",
    "scaUserData": [],
    "accountAccesses": [
        {
            "id": id,
            "iban": iban,
            "currency": "EUR",
            "accessType": "OWNER",
            "scaWeight": 100,
            "accountId": "bZwJMr6GQkguY2IPQqDCWQ" <== we need this value to deposit cash
        }
    ],
    "userRoles": [
        "CUSTOMER"
    ],
    "branch": tpp branch,
    "blocked": false,
    "systemBlocked": false
}
```

### 3.2 Deposit cash
- path : {{host}}:8093/tpp/accounts/{{accountAccess accountId}}/deposit-cash
- POST
- Request
>- Header
>>- Authorization : access_token from response of TPP login
>- path param
>>- {{accountAccess accountId}} : iban access account id (accountId in response of account access information)
>- body (example)
```
{
    "amount": 5000,
    "currency": "EUR"
}
```

- Response (success)
>- Http Status : 202 Accepted

## 4. Transfer
- This case consists of a total of two processes. Creating payment, Authorising payment.

### 4.1 Creating payment
- path : {{host}}:8089/v1/payments/sepa-credit-transfers/
- POST
- Request
>- Header
>>- Accept : application/json
>>- Content-Type : application/json
>>- PSU-ID : YOUR_USER_LOGIN
>>- PSU-IP-Address : 1.1.1.1
>>- X-Request-ID : 2f77a125-aa7a-45c0-b414-cea25a116035
>- body (example)
```
{
    "endToEndIdentification": "TEST",
    "debtorAccount": {
      "currency": "EUR",
      "iban": {{your user iban}} <== iban should match user login you entered in PSU-ID header above
    },
    "instructedAmount": {
      "currency": "EUR",
      "amount": "1000.00"
    },
    "creditorAccount": {
      "currency": "EUR",
      "iban": {{target iban}}
    },
    "creditorAgent" : "AAAADEBBXXX",
    "creditorName": "WBG",
    "creditorAddress": {
      "buildingNumber": "56",
      "townName": "Nürnberg",
      "country": "DE",
      "postCode": "90543",
      "streetName": "WBG Straße"
    },
    "remittanceInformationUnstructured": "Ref. Number WBG-1222"
  }
```

- Response (success)
>- Http Status : 201 Created
>- Body (example)
```
{
    "transactionStatus": "RCVD",
    "paymentId": "KMc1B0b57JMAjLI0u4laGoxFk5HayKRVeXj_upDODyO5PXcJEn9EuEd4Kkwa4-W0cgftJbETkzvNvu5mZQqWcA==_=_psGLvQpt9Q",
    "_links": {
        "updatePsuAuthentication": {
            "href": "http://host:8089/v1/payments/sepa-credit-transfers/KMc1B0b57JMAjLI0u4laGoxFk5HayKRVeXj_upDODyO5PXcJEn9EuEd4Kkwa4-W0cgftJbETkzvNvu5mZQqWcA==_=_psGLvQpt9Q/authorisations/2941c596-6de9-448d-9633-e86c8a130f17"
        },
        "self": {
            "href": "http://host:8089/v1/payments/sepa-credit-transfers/KMc1B0b57JMAjLI0u4laGoxFk5HayKRVeXj_upDODyO5PXcJEn9EuEd4Kkwa4-W0cgftJbETkzvNvu5mZQqWcA==_=_psGLvQpt9Q"
        },
        "status": {
            "href": "http://host:8089/v1/payments/sepa-credit-transfers/KMc1B0b57JMAjLI0u4laGoxFk5HayKRVeXj_upDODyO5PXcJEn9EuEd4Kkwa4-W0cgftJbETkzvNvu5mZQqWcA==_=_psGLvQpt9Q/status"
        },
        "scaStatus": {
            "href": "http://host:8089/v1/payments/sepa-credit-transfers/KMc1B0b57JMAjLI0u4laGoxFk5HayKRVeXj_upDODyO5PXcJEn9EuEd4Kkwa4-W0cgftJbETkzvNvu5mZQqWcA==_=_psGLvQpt9Q/authorisations/2941c596-6de9-448d-9633-e86c8a130f17"
        }
    }
}
```

### 4.2 Authorising payment
- path : response url of Creating payment (_links.updatePsuAuthentication.href)
- PUT
- Request
>- Header
>>- Accept : application/json
>>- Content-Type : application/json
>>- PSU-ID : YOUR_USER_LOGIN
>>- X-Request-ID : 2f77a125-aa7a-45c0-b414-cea25a116035
>- body (example)
```
{
	"psuData": {
		"password": {{your user password}}
	}
}
```

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
{
    "transactionFees": {
        "currency": "EUR",
        "amount": "500"
    },
    "currencyConversionFees": {
        "currency": "EUR",
        "amount": "300"
    },
    "estimatedTotalAmount": {
        "currency": "EUR",
        "amount": "900"
    },
    "estimatedInterbankSettlementAmount": {
        "currency": "EUR",
        "amount": "250"
    },
    "_links": {
        "scaStatus": {
            "href": "http://13.209.50.192:8089/v1/payments/sepa-credit-transfers/KMc1B0b57JMAjLI0u4laGoxFk5HayKRVeXj_upDODyO5PXcJEn9EuEd4Kkwa4-W0cgftJbETkzvNvu5mZQqWcA==_=_psGLvQpt9Q/authorisations/2941c596-6de9-448d-9633-e86c8a130f17"
        }
    },
    "scaStatus": "exempted"
}
```

## 5. Get Account Balance & Transactions
- In this case, access_token must be obtained through banking login in advance.
- If you do not know the user id, you can get the account id through the PSU ID.

### 5.1 Get account access id through PSU Id (Get account balance)
- path : {{host}}:8090/api/v1/ais/accounts/{{PSU ID}}
- GET
- Request
>- Header
>>- Authorization : access_token from response of banking login
>- path param
>>- {{PSU ID}} : PSU ID (user)

- Response (success)
>- Http Status : 200 OK
>- Body (success)
```
[
    {
        "id": "bZwJMr6GQkguY2IPQqDCWQ", <== this value needed to query transaction
        "iban": "GB23YKGU10834700000115",
        "currency": "EUR",
        "name": "tester1",
        "accountType": "CASH",
        "accountStatus": "ENABLED",
        "linkedAccounts": "D1-iUWmBTLktJ08PoExxnU",
        "usageType": "PRIV",
        "balances": [
            {
                "amount": {
                    "currency": "EUR",
                    "amount": 4000.00
                },
                "balanceType": "INTERIM_AVAILABLE",
                "lastChangeDateTime": "2021-08-06T06:49:54.837",
                "referenceDate": "2021-08-06"
            },
            {
                "amount": {
                    "currency": "EUR",
                    "amount": 4000.00
                },
                "balanceType": "CLOSING_BOOKED",
                "lastChangeDateTime": "2021-08-06T06:49:54.837",
                "referenceDate": "2021-08-06"
            }
        ],
        "blocked": false,
        "systemBlocked": false,
        "creditLimit": 0.00,
        "branch": "GB_YKGU"
    }
]
```

### 5.2 Get account transaction
- path : {{host}}:8090/api/v1/ais/transactions/{{user id}}?dateFrom=2021-06-20&dateTo=2021-06-25
- GET
- Request
>- Header
>>- Authorization : access_token from response of banking login
>- path param
>>- {{user id}} : id in response of Get account access id through PSU Id
>- param
>>- dateFrom : query start date 
>>- dateTo : query end date

- Response (success)
>- Http Status : 200 OK
>- Body (success)
```
[
    {
        "transactionId": "DK3bMtMGSMEp6jaGqvKT4Q",
        "endToEndId": "DK3bMtMGSMEp6jaGqvKT4Q",
        "bookingDate": "2021-08-06",
        "valueDate": "2021-08-06",
        "amount": {
            "currency": "EUR",
            "amount": 5000
        },
        "creditorName": "tester1",
        "creditorAccount": {
            "iban": "GB23YKGU10834700000115",
            "currency": "EUR"
        },
        "debtorName": "tester1",
        "debtorAccount": {
            "iban": "GB23YKGU10834700000115",
            "currency": "EUR"
        },
        "remittanceInformationUnstructured": "Cash deposit through Bank ATM",
        "bankTransactionCode": "PMNT-MCOP-OTHR",
        "proprietaryBankTransactionCode": "PMNT-MCOP-OTHR",
        "balanceAfterTransaction": {
            "amount": {
                "currency": "EUR",
                "amount": 5000
            },
            "balanceType": "INTERIM_AVAILABLE",
            "lastChangeDateTime": "2021-08-06T05:31:29.144427",
            "referenceDate": "2021-08-06"
        }
    }
]
```

