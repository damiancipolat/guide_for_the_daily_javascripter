<img src="https://github.com/damcipolat-recargapay/js_developer_guide/blob/master/img/recarga.jpg?raw=true" width="250px" align="right" />

# Code review
In this document there are several cases of code that can be improved or refactored, to improve quality and apply good practices.


| Repository | Refactors |
| ------------- | ------------- |
| recarga-utilities-refunds  | 1 to 6 |
| recarga-orders  | 7 to 9  |


### Refactor 1:
> File location: https://github.com/recarga/recarga-utilities-refunds/blob/staging/app/config.js

#### Problems detected:

 - Executable code out of function, all the executable code in a module should be run into a function.
 - Use let in variables that the content will be modified in this case "customProperties".

#### **Current code**:

 ```javascript
 const PROPERTY_PREFIX = 'rur_';

 const customProperties = {};

 for (const key in process.env) {
   if (key && key.startsWith(PROPERTY_PREFIX) && process.env[key]) {
     const value = process.env[key];
     const propertyName = key.substr(PROPERTY_PREFIX.length);
     customProperties[propertyName] = value;
   }
 }

 module.exports = {
   ...customProperties
 }
 ```
    
#### **Refactored code**:

 ```javascript
 const PROPERTY_PREFIX = 'rur_';

 const loadProperties = ()=>{

     let customProperties = {};

     for (const key in process.env) {
       if (key && key.startsWith(PROPERTY_PREFIX) && process.env[key]) {
         const value = process.env[key];
         const propertyName = key.substr(PROPERTY_PREFIX.length);
         customProperties[propertyName] = value;
       }
     }

     return customProperties;
 }

 module.exports = {
  ...loadProperties()
 }
 ```

### Refactor 2:
> File location: https://github.com/recarga/recarga-utilities-refunds/blob/staging/app/secretConfig.js

#### Problems detected:

 - Executable code out of function, apply refactor 1.
 - Use assert to validate type in more relevant functions.
 - If is possible use arrow function to write less code.
 - Innecesary async in decrypt, and .promise for aws-sdk code missing.

#### **Current code**:

 ```javascript
 function requireDecrypt(value) {
  assert(value, 'value shouldn\'t be null or undefined');
  return typeof value === 'string' && value.startsWith(CIPHER_PREFIX);
}

async function decrypt(value) {
  assert(value, 'value shouldn\'t be null or undefined');
  const kms = new awsSdk.KMS();
  return kms.decrypt(
    {
      CiphertextBlob: Buffer.from(value.substr(CIPHER_PREFIX.length), 'base64')
    });
}

 ```
    
#### **Refactored code**:

 ```javascript
 const requireDecrypt = (value)=> value&&typeof value ==='string'&&value.startsWith(CIPHER_PREFIX);

 function decrypt(value) {
    assert(value, 'value shouldn\'t be null or undefined');
    const kms = new awsSdk.KMS();
    return kms.decrypt({
     CiphertextBlob: Buffer.from(value.substr(CIPHER_PREFIX.length), 'base64')
    }).promise();
 }
 ```
 
### Refactor 3:
> File location: https://github.com/recarga/recarga-utilities-refunds/blob/staging/app/services/reportExampleService.js

#### Problems detected:

 - Avoid Try / catch for logging purposes, if is necessary make this in a controller. 

#### **Current code**:

 ```javascript
   async function generateData(startDate, endDate) {
     assert(startDate && endDate, 'Required parameters');
     assert(typeof startDate === 'number', 'startDate should be a number (date timestamp)');
     assert(typeof endDate === 'number', 'endDate should be a number (date timestamp)');

     var data;
     try {
       data = await readFilePromise(path.join(__dirname, '../celcoin-refunds-response-template.json'), { encoding: 'utf-8' });
     } catch (err) {
       console.info('there was a problem reading file: ' + err);
       throw err;
     }
     return data.replace(DATE_REPLACER_REGEX, new Date(secureRandom() * (endDate - startDate) + startDate).toISOString());
   }
 ```
    
#### **Refactored code**:

 ```javascript
  async function generateData(startDate, endDate) {
    assert(startDate && endDate, 'Required parameters');
    assert(typeof startDate === 'number', 'startDate should be a number (date timestamp)');
    assert(typeof endDate === 'number', 'endDate should be a number (date timestamp)');

    const data = await readFilePromise(path.join(__dirname, '../celcoin-refunds-response-template.json'), { encoding: 'utf-8' });     
    return data.replace(DATE_REPLACER_REGEX, new Date(secureRandom() * (endDate - startDate) + startDate).toISOString());
  }
 ```
 
### Refactor 4:
> File location: https://github.com/recarga/recarga-utilities-refunds/blob/staging/app/services/reportDownloader.js

#### Problems detected:

 - Throw a new Error('message') instead of a string.
 - Compare with constants.

#### **Current code**:

 ```javascript
async function download(startDate, endDate) {
  assert(startDate && endDate, 'Required parameters');
  assert(typeof startDate === 'object', 'startDate should be an object');
  assert(typeof endDate === 'object', 'endDate should be an object');

  const url = config.host.concat(config.service
    .replace('$$START_DATE$$', formatYYYYMMDD(startDate))
    .replace('$$END_DATE$$', formatYYYYMMDD(endDate)));
  console.info('formed url: %s', url);
  const values = await Promise.all([secretConfig.username, secretConfig.password]);
  const response = await unirest.get(url)
    .auth({
      user: values[0],
      pass: values[1],
      sendImmediately: true
    })
    .header('Accept', 'application/json')
    .send();
  if (response.statusCode != 200) {
    console.error('Got an %s HTTPs status. Error: %o. Response body: "%o"', response.statusCode, response.error, response.body);
    throw response.error; //<----
  } else {
    return response.body;
  }
}
 ```
    
#### **Refactored code**:

 ```javascript
  const SUCCESS_DOWNLOAD = 200;
 
  if (response.statusCode != SUCCESS_DOWNLOAD){

    console.error('Got an %s HTTPs status. Error: %o. Response body: "%o"', response.statusCode, response.error, response.body);
    throw new Error(response.error);

  } else {
    return response.body;
  }
 ```
 
### Refactor 5:
> File location: https://github.com/recarga/recarga-utilities-refunds/blob/staging/app/controllers/reportExample.js

#### Problems detected:

 - Avoid if / else whenever possible.
 - Launch custom exception objects.
 - Avoid nested conditions, is a good idea to validate the wrong case first to launch exceptions.

#### **Current code**:

 ```javascript
async function download(startDate, endDate) {
  assert(startDate && endDate, 'Required parameters');
  assert(typeof startDate === 'object', 'startDate should be an object');
  assert(typeof endDate === 'object', 'endDate should be an object');

  const url = config.host.concat(config.service
    .replace('$$START_DATE$$', formatYYYYMMDD(startDate))
    .replace('$$END_DATE$$', formatYYYYMMDD(endDate)));
  console.info('formed url: %s', url);
  const values = await Promise.all([secretConfig.username, secretConfig.password]);
  const response = await unirest.get(url)
    .auth({
      user: values[0],
      pass: values[1],
      sendImmediately: true
    })
    .header('Accept', 'application/json')
    .send();
  if (response.statusCode != 200) {
    console.error('Got an %s HTTPs status. Error: %o. Response body: "%o"', response.statusCode, response.error, response.body);
    throw response.error; //<----
  } else {
    return response.body;
  }
}
 ```
    
#### **Refactored code**:

 ```javascript
async function get(event) {

  try{

    if (!event.queryStringParameters || !event.queryStringParameters['DataInicio'] || !event.queryStringParameters['DataFim'])
      throw {statusCode:400, detail:'Missing Parameters'};


    if (event.queryStringParameters['failWithStatusCode'])
      throw {statusCode:event.queryStringParameters['failWithStatusCode'], detail:'failed with statusCode: ' + event.queryStringParameters['failWithStatusCode']};

    const startDate = Date.parse(event.queryStringParameters['DataInicio']);
    const endDate   = Date.parse(event.queryStringParameters['DataFim']);

    if !(startDate && endDate && !isNaN(startDate) && !isNaN(endDate))
      throw {statusCode: 400, body: 'wrong parameters'};

    const data = await celcoinReportExampleService.generateData(startDate, endDate);

    return {
      statusCode: 200,
      body: data
    };

  } catch(error){
    console.error('There are problems processing the download', error);
    
    return {
      error: err.statusCode||500,
      body: err.detail||'Unknow error'
    };
  }

}

module.exports = {
  get
};
 ```

 ### Refactor 6:
> File location: https://github.com/recarga/recarga-utilities-refunds/blob/staging/app/__tests__/services/csvBuilder.test.js

#### Problems detected:

 - Don't throw an exception within a test, the function tested is the one who should throw them
 - Avoid use `try/catch` into a unit test, use the jest functions to handle promise reject or exceptions. 
   Promise.reject expect(xxxx).rejects.toMatch('errror...').
   For function exceptions, expect(function).toThrow();

#### **Current code**:

 ```javascript
  test('fails with no parameters', () => {
    try {
      csvBuilder.build();
      throw 'Not expected to get here!';
    } catch (error) {
      expect(error.message).toBe('Required parameters')
    }
  });
 ```
    
#### **Refactored code**:

 ```javascript
   test('fails with no parameters', () => {
       expect(csvBuilder.build).toThrow();
  });
 ```

 ### Refactor 7:
> File location: https://github.com/recarga/recarga-orders/blob/master/api/services/order-post-gen.js

#### Problems detected:

 - Use `const` for all the imported modules.
 - Avoid harcoded variables (retryPeriod ), get the values from a config file, declare them using `const`.
 - Don't Mix async/await with promise.then.
 - Finish the lines with a semicolon.
 - Avoid parameters mutation, orderuuid.
 - Make only one `module.exports`.
 - Unnecesary async in function if the returns is a aws-sdk.promise() object.
 
#### **Current code**:

 ```javascript
let dynamodb = require('serverless-dynamodb-client')
let retryPeriod = 500

module.exports.enqueue = async (request, orderid, orderuuid) => {
  orderuuid = orderuuid || uuid()
  console.info('request queued (', orderuuid, ', ', orderid, ') - ', request)

  request.orderuuid = orderuuid
  await docClient
    .put({
      TableName: queuedOrderName,
      Item: {
        orderuuid: orderuuid,
        orderid: orderid,
        creationDate: Date.now(),
        request: request
      },
      ReturnValues: 'NONE',
      ReturnConsumedCapacity: 'TOTAL',
      ReturnItemCollectionMetrics: 'SIZE'
    })
    .promise()
    .then(data => console.info('Order ', orderid, 'enqueued - ', data))

  await sqs
    .sendMessage({
      MessageBody: orderuuid,
      QueueUrl: orderQueue
    })
    .promise()

  return { status: 'queue', orderuuid: orderuuid }
}

 ```
    
#### **Refactored code**:

 ```javascript
const dynamodb = require('serverless-dynamodb-client');
const retryPeriod = 500;

const enqueue = async (request, orderid, orderuuid) => {

  const uuid  = orderuuid || uuid();  
  console.info('request queued (', uuid, ', ', orderid, ') - ', request)

  const newRequest = {
    ..request,
    orderuuid
  };
  
  const data = await docClient
    .put({
      TableName: queuedOrderName,
      Item: {
        orderuuid: orderuuid,
        orderid: orderid,
        creationDate: Date.now(),
        request: newRequest
      },
      ReturnValues: 'NONE',
      ReturnConsumedCapacity: 'TOTAL',
      ReturnItemCollectionMetrics: 'SIZE'
    })
    .promise();

  console.info('Order ', orderid, 'enqueued - ', data);

  await sqs
    .sendMessage({
      MessageBody: orderuuid,
      QueueUrl: orderQueue
    })
    .promise();

  return { 
    status: 'queue', 
    orderuuid: orderuuid 
  };

}

module.exports = {
 enqueue
};
 ```

### Refactor 8:WIP
> File location: https://github.com/recarga/recarga-orders/blob/master/api/lambdas/order-post/ordersController.js

#### Problems detected:

 - Use `const` for all the imported modules.
 - Avoid to use delete.
 - Avoid mutations with parameters variables, in this case (event).
 - Finish the lines with a semicolon.

#### **Current code**:

 ```javascript
let unirest = require('unirest')
let service = require('../../services/order-post-gen')

module.exports.post = async event =>
  httpTimeout(async _ => {
    const start = Date.now()
    delete event.headers.Host
    delete event.headers.host
    event.headers['X-Proxyed-Order'] = 'true'
    event.headers['X-Forwarded-For-Bak'] = event.headers['X-Forwarded-For']
    event.headers['X-Forwarded-For'] = event.headers['X-Real-IP']
    console.info('headers=', event.headers)
    const req = { headers: event.headers, body: event.body }
    const body = bodyDecode(event.body)
    const orderQueued = await service.enqueue(req, body['shopping_cart_id'], body['shopping_cart_uuid'])
    const orderuuid = orderQueued.orderuuid

    const response = (await service.getResponseToOrder(orderuuid)).response
    response.headers['X-Proxyed-Order'] = true
    if (response.error) {
      console.error('Error processing Order POST: ', response.error, 'RESPONSE - ', {
        httpStatus: response.statusCode || 500,
        reqDuration: response.duration,
        duration: Date.now() - start
      })
      return {
        statusCode: response.statusCode || 500,
        body: JSON.stringify(response.body) || JSON.stringify({ error: 'internal server error' })
      }
    } else {
      const body = response.body
      body.proxied = process.env.ENV_SERVER

      body.orderuuid = orderuuid
      console.info('SUCCESS RESPONSE - ', {
        httpStatus: response.statusCode,
        orderStatus: body.status,
        orderMainCardType: body.mainCardType,
        userType: body.userType,
        orderAmountUSD: body.amountInDollars,
        orderId: body.orderId,
        reqDuration: response.duration,
        duration: Date.now() - start,
        orderuuid: orderuuid
      })
      console.debug('resp heades=', response.headers)
      delete response.headers['Content-Length']
      delete response.headers['content-length']
      delete response.headers['Content-Type']
      delete response.headers['content-type']
      delete response.headers['Content-Encoding']
      delete response.headers['content-encoding']
      delete response.headers['Transfer-Encoding']
      delete response.headers['transfer-encoding']
      return {
        statusCode: response.statusCode,
        headers: response.headers,
        body: JSON.stringify(body)
      }
    }
  })

 ```
    
#### **Refactored code**:

 ```javascript
const unirest = require('unirest');
const service = require('../../services/order-post-gen');
 
const post = async event =>
  httpTimeout(async _ => {
    const start = Date.now();

    const paramEvent = {
      ..event,
      'X-Proxyed-Order':true,
      'X-Forwarded-For-Bak':event.headers['X-Forwarded-For'],
      'X-Forwarded-For':event.headers['X-Real-IP']
    };

    console.info('headers=', event.headers);

    const req = { 
      headers: paramEvent.headers, 
      body: paramEvent.body 
    };

    const body = bodyDecode(paramEvent.body);
    const orderQueued = await service.enqueue(req, body['shopping_cart_id'], body['shopping_cart_uuid']);
    const orderuuid = orderQueued.orderuuid;

    const response = (await service.getResponseToOrder(orderuuid)).response;

    response.headers['X-Proxyed-Order'] = true
    if (response.error) {
      console.error('Error processing Order POST: ', response.error, 'RESPONSE - ', {
        httpStatus: response.statusCode || 500,
        reqDuration: response.duration,
        duration: Date.now() - start
      })
      return {
        statusCode: response.statusCode || 500,
        body: JSON.stringify(response.body) || JSON.stringify({ error: 'internal server error' })
      }
    } else {
      const body = response.body
      body.proxied = process.env.ENV_SERVER

      body.orderuuid = orderuuid
      console.info('SUCCESS RESPONSE - ', {
        httpStatus: response.statusCode,
        orderStatus: body.status,
        orderMainCardType: body.mainCardType,
        userType: body.userType,
        orderAmountUSD: body.amountInDollars,
        orderId: body.orderId,
        reqDuration: response.duration,
        duration: Date.now() - start,
        orderuuid: orderuuid
      })
      console.debug('resp heades=', response.headers)
      delete response.headers['Content-Length']
      delete response.headers['content-length']
      delete response.headers['Content-Type']
      delete response.headers['content-type']
      delete response.headers['Content-Encoding']
      delete response.headers['content-encoding']
      delete response.headers['Transfer-Encoding']
      delete response.headers['transfer-encoding']
      return {
        statusCode: response.statusCode,
        headers: response.headers,
        body: JSON.stringify(body)
      }
    }
  })

module.exports = {
  post
};
 ```
 
### Refactor 9:
> File location: https://github.com/recarga/recarga-orders/blob/master/api/lambdas/order-consumer/ordersConsumer.js

#### Problems detected:

 - Avoid use async, when the function return a promise. (Promise.all).
 - Try to avoid nested if.

#### **Current code**:

 ```javascript
module.exports.consumeSQS = async event => {
  console.debug('SQS event=', event)
  return Promise.all(
    event.Records.map(async recEvent => {
      console.debug('SQS eachEvent=', recEvent)
      const orderuuid = recEvent.body
      console.info('SQS orderuuid=', orderuuid)
      if (recEvent.attributes.SentTimestamp > Date.now() - 60000) {
        await consume(orderuuid)
      } else {
        discard(orderuuid, 'Order expired in queue')
      }
      await sqs
        .deleteMessage({
          QueueUrl: orderQueue,
          ReceiptHandle: recEvent.receiptHandle
        })
        .promise()
    })
  )
}
 ```
    
#### **Refactored code**:

 ```javascript
const consumeSQS = async event => {
  console.debug('SQS event=', event);

  return Promise.all(
    event.Records.map(async recEvent => {
      console.debug('SQS eachEvent=', recEvent);
      const orderuuid = recEvent.body;
      console.info('SQS orderuuid=', orderuuid);
      
      if (recEvent.attributes.SentTimestamp > Date.now() - 60000)
        await consume(orderuuid);
      else
        discard(orderuuid, 'Order expired in queue');

      await sqs
        .deleteMessage({
          QueueUrl: orderQueue,
          ReceiptHandle: recEvent.receiptHandle
        })
        .promise();
    })
  );

}

const ERROR_STATUS = 503;

const consume = async (orderuuid) =>{

  const start = Date.now();
  console.info('new request for:', orderuuid);
  const orderReq = await service.countTries(orderuuid);
  console.info('order marked =', orderReq);

  if (orderReq.requestTried > 3){
    discard(orderuuid, 'Too many tryies');
    return null;
  }
    
  if (orderReq.response){
    discard(orderuuid, 'Order already processed');
    return null;
  }

  const request = orderReq.request;
  const url = `${protocol}://${domain}/${resource}`;

  console.debug('No response, sending headers =', request.headers, 'body=', request.body, 'to =', url)
      
  const responsePromise = await unirest
    .post(url)
    .pool(pool)
    .headers(request.headers)
    .timeout(30000)
    .send(request.body);
  
  console.debug('Repost sent');

  const response = await responsePromise;
  const body = response.body;
  console.debug('Repost response=', response.body);

  if (response.statusCode === ERROR_STATUS) {

    console.error(
      'ERROR - Server unavailable: ',
      response.error,
      'httpStatus=',
      response.statusCode,
      'holdON for ',
      holdPeriod,
      'msec'
    );

    await sleep(holdPeriod);
    throw 'TS unavailable holding on - ' + response.error;

  }

  const result = await service.updateOrder(orderuuid, {
    headers: response.headers,
    body: body,
    duration: Date.now() - start,
    error: response.error,
    statusCode: response.statusCode
  });

  return result;

}


module.exports = {
  consumeSQS,
  consume
};
 ```

### Refactor 10:
> File location: https://github.com/recarga/recarga-orders/blob/master/api/__tests__/order-consumer.test.js

#### Problems detected:

 - Avoid using external variables to record test changes, use jest call logs with a spyOn.

#### **Current code**:

 ```javascript
it('can consume a SQS event', () => {
    let consumedOrders = []
    let postResult = {}
    mod.__set__('service', {
      countTries: async orderuuid => {
        consumedOrders.push(orderuuid)
        return {
          mocked: 'mock',
          requestTried: 1,
          request: {
            headers: {},
            body: 'mock'
          }
        }
      },
      updateOrder: async (orderuuid, response) => {
        postResult.response = response
        return {}
      }
    })
    return mod
      .consumeSQS({
        Records: [....]
      })
      .then(res => {
.....
      })
  })
 ```
    
#### **Refactored code**:

 ```javascript
  it('can consume a SQS event', async () => {

    const service = require('../services/order-post-gen');
    const mod     = require('./../lambdas/order-consumer/ordersConsumer');
   
    const countriesSpy = jest.spyOn(service, 'countTries').mockImplementation(async orderuuid => {
      return {
        mocked: 'mock',
        requestTried: 1,
        request: {
          headers: {},
          body: 'mock'
        }
      };
    });
    
    const updateOrderSpy = jest.spyOn(service, 'updateOrder').mockImplementation(async (orderuuid, response) => {
      return {}
    });

    const unirest = require('unirest');

    jest.mock('unirest', () => {
      const original = jest.requireActual('unirest');
      return {
        ...original,
        post:()=>({
          pool:()=>({
            headers:()=>({
              timeout:()=>({
                send:()=>Promise.resolve({
                  headers: { mock: 'mock' },
                  statusCode: 200,
                  body: { backendResp: 'mocked' }
                })
              })
            })
          })
        })
      }
    });

    await mod.consumeSQS({
      Records: [
        {
          body: 'mockedUUID',
          receiptHandle: 'mockedReceipt',
          attributes: {
            SentTimestamp: Date.now()
          }
        }
      ]
    });

    expect(countriesSpy).toBeCalled();
    expect(countriesSpy.mock.calls[0][0]).toBe('mockedUUID');
    expect(updateOrderSpy).toBeCalled();
    expect(updateOrderSpy.mock.calls[0][1].body).toBeDefined();
    expect(updateOrderSpy.mock.calls[0][1].body.backendResp).toBe('mocked');
    expect(updateOrderSpy.mock.calls[0][1].headers).toBeDefined();
    expect(updateOrderSpy.mock.calls[0][1].headers.mock).toBe('mock');

  });
 ```
