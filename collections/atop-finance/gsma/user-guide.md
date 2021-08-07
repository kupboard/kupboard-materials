# GSMA API Guide
- The Mobile Money API is an initiative developed through collaboration between the mobile money industry and the GSMA, which provides a harmonized API Specification for all the common mobile money use cases which is both easy to use and secure.
- This api set was generated through openapi generator.
- Currently, separate API authentication is not supported.
- The list of all APIs can be checked directly through swagger.
>- Host:8060/swagger-ui.html

## Use Cases
- 1. Create User
- 2. Get Account Balance & Transactions
- 3. Transfer account to account

## 1. Create user

### 1.1 Create user
- path : {{host}}:8060/simulator/v1.2/passthrough/mm/accounts/individual
- POST
- Request
>- Body (example)
```
{
  "accountIdentifiers": [
    {
      "key": "msisdn",
      "value": "+821011112222"
    }
  ],
  "identity": [
    {
      "identityKyc": {
        "birthCountry": "AD",
        "contactPhone": "+821011112222",
        "dateOfBirth": "1988-11-20",
        "emailAddress": "user1@gmail.com",
        "employerName": "string",
        "gender": "m",
        "idDocument": [
          {
            "idType": "passport",
            "idNumber": "111111",
            "issueDate": "2018-11-20",
            "expiryDate": "2018-11-20",
            "issuer": "ABC",
            "issuerPlace": "DEF",
            "issuerCountry": "AD"
          }
        ],
        "nationality": "AD",
        "occupation": "Miner",
        "postalAddress": {
          "addressLine1": "37",
          "addressLine2": "ABC Drive",
          "addressLine3": "string",
          "city": "Berlin",
          "stateProvince": "string",
          "postalCode": "AF1234",
          "country": "AD"
        },
        "subjectName": {
          "title": "Mr",
          "firstName": "tester1",
          "middleName": "tester1",
          "lastName": "tester1",
          "fullName": "tester1",
          "nativeName": "tester1"
        }
      },
      "accountRelationship": "accountholder",
      "kycVerificationStatus": "verified",
      "kycVerificationEntity": "ABC Agent",
      "kycLevel": 1,
      "customData": [
        {
          "key": "test",
          "value": "custom"
        }
      ]
    }
  ],
  "accountType": "string",
  "customData": [
    {
      "key": "test",
      "value": "custom1"
    }
  ],
  "fees": [
    {
      "feeType": "string",
      "feeAmount": "1.46",
      "feeCurrency": "RWF"
    }
  ],
  "registeringEntity": "ABC Agent",
  "requestDate": "2021-06-09T15:41:45.194Z"
}
```

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
{
    "accountIdentifiers": [
        {
            "key": "msisdn",
            "value": "+821011112222"
        }
    ],
    "identity": [
        {
            "identityId": "identityId",
            "identityType": "individual",
            "identityKyc": {
                "contactPhone": "+821011112222",
                "emailAddress": "user1@gmail.com",
                "idDocument": [
                    {
                        "idType": "passport",
            		"idNumber": "111111",
            		"issueDate": "2018-11-20",
            		"expiryDate": "2018-11-20",
            		"issuer": "ABC",
            		"issuerPlace": "DEF",
            		"issuerCountry": "AD"
                    }
                ],
                "subjectName": {
                     "title": "Mr",
         	     "firstName": "tester1",
          	     "middleName": "tester1",
          	     "lastName": "tester1",
          	     "fullName": "tester1",
          	     "nativeName": "tester1"
                }
            },
            "accountRelationship": "accountholder"
        }
    ],
    "accountStatus": "available",
    "creationDate": "2021-08-06T06:41:15.404Z"
}
```

## 2. Get Account Balance & Transactions

### 2.1 Get Account Balance
- path : {{host}}:8060/simulator/v1.2/passthrough/mm/accounts/{{account identifier type}}/{{account identifier value}}/balance
- GET
- Request
>- path param
>>- {{account identifier type}} : used account identifier type when creating user (like msisdn)
>>- {{account identifier value}} : account identifier value

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
{
    "accountStatus": "available",
    "currentBalance": "0.00",
    "availableBalance": "0.00",
    "reservedBalance": "0.00",
    "unclearedBalance": "0.00",
    "currency": "AED"
}
```

### 2.2 Get Account Transaction
- path : {{host}}:8060/simulator/v1.2/passthrough/mm/accounts/{{account identifier type}}/{{account identifier value}}/transactions?offset=0&limit=20
- GET
- Request
>- path param
>>- {{account identifier type}} : used account identifier type when creating user (like msisdn)
>>- {{account identifier value}} : account identifier value
>- param
>>- offset : pagination start point
>>- limit : pagination size

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
{
    "transactions": [
        {
            "transactionReference": "Tr6",
            "creditParty": [
                {
                    "key": "msisdn",
                    "value": "+821011112222"
                }
            ],
            "debitParty": [
                {
                    "key": "msisdn",
                    "value": "+821033334444"
                }
            ],
            "type": "transfer",
            "transactionStatus": "executed",
            "amount": "1",
            "currency": "GBP",
            "descriptionText": "",
            "creationDate": "2021-07-15T07:50:18.312"
        }
    ]
}
```

## 3. Transfer account to account
- When a transaction occurs, the banking module checks the transaction every 5 seconds and changes the state.

### 3.1 Get account holder info
- path : {{host}}:8060/simulator/v1.2/passthrough/mm/accounts/{{account identifier type}}/{{account identifier value}}/accountname
- GET
- Request
>- path param
>>- {{account identifier type}} : used account identifier type when creating user (like msisdn)
>>- {{account identifier value}} : account identifier value

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
{
    "name": {
        "title": "Mr",
        "firstName": "David",
        "middleName": "Robert Joseph",
        "lastName": "Beckham",
        "fullName": "David Robert Joseph Beckham",
        "nativeName": "David Robert Joseph Beckham"
    },
    "lei": ""
}
```

### 3.2 Transfer 
- path : {{host}}:8060/simulator/v1.2/passthrough/mm/transactions/type/transfer
- POST
- Request
>- Body (example)
```
{
    "amount": "40.00",
    "creditParty": [
        {
            "key": "msisdn",
            "value": "+821011112222"
        }
    ],
    "currency": "AED",
    "debitParty": [
        {
            "key": "msisdn",
            "value": "+821033334444"
        }
    ],
  "requestingOrganisation": {
    "requestingOrganisationIdentifierType": "organisationid",
    "requestingOrganisationIdentifier": "testorganisation"
  }
}
```

- Response (success)
>- Http Status : 200 OK
>- Body (example)
```
{
    "transactionReference": "Tr9",
    "type": "transfer",
    "transactionStatus": "pending",
    "amount": "40.00",
    "currency": "AED",
    "creationDate": "2021-08-06T07:10:20.082Z",
    "creditParty": [],
    "debitParty": []
}
```