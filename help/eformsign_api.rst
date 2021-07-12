
--------------------------
eformsign API 사용하기
--------------------------

eformsign이 제공하는 API를 사용해 eformsign의 기능을 고객의 시스템/서비스에서 호출해 사용할 수 있도록 해주는 기능입니다.


시작하기 
=========

eformsign API를 사용하기 위해서는 다음의 준비 작업이 필요합니다.

- 회사 ID와 문서 ID 확인하기
- API 키 발급 및 비밀키 확인하기
- 서명 생성하기

.. caution:: 
   
   서명 생성에는 30초의 시간 제한이 있습니다. 30초 이내에 서명을 생성하고 토큰을 발급받아야 합니다. 



회사 ID와 문서 ID 확인하기
---------------------------

eformsign API를 사용하기 위해서는 소속 회사의 ID와 조회하고자 하는 문서의 ID를 알고 있어야 합니다. 

eformsign 서비스에 로그인하여 회사 ID와 문서 ID를 확인해 주십시오.

.. note:: 

   회사 ID는 왼쪽 메뉴 트리의 회사 관리 > 회사 정보 메뉴의 "기본 정보" 탭에서 확인할 수 있습니다.

   |image1|

   문서 ID는 각 문서함 오른쪽 상단의 문서 아이콘(|image2|)을 클릭하여 문서 ID 컬럼을 추가하면 조회하고자 하는 문서의 ID를 확인할 수 있습니다. 

   |image3|



.. _apikey:

API 키 발급 및 비밀키 확인하기
------------------------------

1. eformsign에 대표 관리자로 로그인 후, 메뉴 트리에서 **[커넥트] > [API / Webhook]** 페이지로 이동합니다. 

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: 커넥트 > API/Webhook 메뉴 위치


2. **[API 키 관리]** 탭을 선택하고 **API 키 생성** 버튼을 클릭합니다.

.. image:: resources/apikey2.PNG
    :width: 700
    :alt: API 키 생성 버튼


3. API 키 생성 팝업창에 별칭과 애플리케이션 이름을 입력하고 저장 버튼을 클릭합니다.

.. image:: resources/apikey3.PNG
    :width: 700
    :alt: API 키 생성 팝업창


4. 생성된 키 목록에서 **키보기** 버튼을 클릭하여 API 키와 비밀 키를 확인합니다.

.. image:: resources/apikey4.PNG
    :width: 700
    :alt: API 키보기 버튼 위치

.. image:: resources/apikey5.PNG
    :width: 700
    :alt: API 키 및 비밀 키 확인 



.. note:: **API 키 수정 방법**

    생성된 키 목록에서 **수정** 버튼을 클릭하면 별칭과 어플리케이션 이름을 수정할 수 있습니다. 또한, 상태 영역을 클릭하여 활성/비활성 상태로 변경할 수도 있습니다.

.. note:: **API 키 삭제 방법**

    생성된 키 목록에서 **삭제** 버튼을 클릭하여 API 키를 삭제할 수 있습니다.


서명 생성하기 
==============

eformsign_signature는 비대칭 키 방식과 타원곡선 암호화(Elliptic curve cryptography)를 사용하고 있습니다.

.. tip:: 
   
   타원곡선 암호화는 공개키 암호화 방식 중 하나로, 데이터 암호화 디지털 인증 등 현재 가장 많이 쓰이는 암호방식입니다. 


서명 생성 방법에 대해서는 Java, Python, PHP 언어별로 설명합니다.

Java
-------

서버의 현재 시간을 String(UTF-8)으로 변환하고, `API Key 발급하기 <#apikey>`__\에서 발급받은 private key로 서명한 후, 서명한 데이터를 hex string으로 변환합니다.

.. note:: 

   서명 알고리즘은 SHA256withECDSA을 사용합니다.


Javascript(Node.JS)
------------------------

Jsrsasign(https://kjur.github.io/jsrsasign/) npm 종속성이 설치되어 있어야 합니다.

.. code:: Javascript

   npm install jsrsasign



Python
-------

키 포맷 처리용 라이브러리를 사용해야 합니다. 작업전 다음의 명령어를 통해 해당 라이브러리를 설치해 주세요.

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

PHP 예제를 사용하려면 PHP OpenSSL 라이브러리가 설치되어 있어야 하며, 다음 예제의 keycheck.inc.php, test.php 파일이 동일한 패스에 위치하게 한 후에 진행해야 합니다. 


각 언어별 예제
---------------------

다음은 각 언어별 예제입니다.


.. note:: 

   execution_time은 long 타입을 사용합니다. 따라서, execution_time 입력 시에는 Access Token 발급 시에 확인한 시간 뒤에 L을 넣어 입력합니다.  



.. code-tabs::

    .. code-tab:: java
        :title: Java

        import java.security.KeyFactory;
        import java.security.spec.PKCS8EncodedKeySpec;
        import java.security.PrivateKey;
        import java.security.Signature;
         
        //private key
        String privateKeyHexStr = "발급 받은 private key(String)";    //회사 커넥트 > API Key의 Private key 값을 넣어야 함
        KeyFactory keyFact = KeyFactory.getInstance("EC");
        PKCS8EncodedKeySpec psks8KeySpec = new PKCS8EncodedKeySpec(new BigInteger(privateKeyHexStr,16).toByteArray());
        PrivateKey privateKey = keyFact.generatePrivate(psks8KeySpec);
         
        //execution_time - 서버 현재 시간
        //long execution_time = new Date().getTime();
        long execution_time = 1611537340731L;     //Access_token 발급 시에 취득한 execute_time을 여기에 입력. long 값이기 때문에 받은 시간 뒤에 L 추가     
        String execution_time_str = String.valueOf(execution_time);
         
        //eformsign_signature 생성
        Signature ecdsa = Signature.getInstance("SHA256withECDSA");
        ecdsa.initSign(privateKey);
        ecdsa.update(execution_time_str.getBytes("UTF-8"));
        String eformsign_signature = new BigInteger(ecdsa.sign()).toString(16);
         
         
        //현재 시간 및 현재 시간 서명값
        System.out.print("execution_time : "+execution_time);
        System.out.print("eformsign_signature : "+eformsign_signature);

    .. code-tab:: javascript
        :title: Javascript(Node.JS)

        const rs = require('jsrsasign');


        // User-Data-Here
        const execution_time  = Date.now()+"";
        const privateKeyHex = "발급 받은 private key(String)";

        // User-Data-Here
        var privateKey = rs.KEYUTIL.getKeyFromPlainPrivatePKCS8Hex(privateKeyHex);

        // Sign
        var s_sig = new rs.Signature({alg: 'SHA256withECDSA'});
        s_sig.init(privateKey);
        s_sig.updateString(execution_time);
        var signature = s_sig.sign();
        console.log('data:', execution_time);
        console.log('eformsign_signature:', signature);


    .. code-tab:: python
        :title: Python

        import hashlib
        import binascii
         
        from time import time
        from ecdsa import SigningKey, VerifyingKey, BadSignatureError
        from ecdsa.util import sigencode_der, sigdecode_der
         
        # private key
        privateKeyHex = "발급 받은 private key(String)"
        privateKey = SigningKey.from_der(binascii.unhexlify(privateKeyHex))
         
        # execution_time - 서버 현재 시간
        execution_time_int = int(time() * 1000)
        execution_time = str(execution_time_int)
          
        # eformsign_signature 생성
        eformsign_signature = privateKey.sign(execution_time.encode('utf-8'), hashfunc=hashlib.sha256, sigencode=sigencode_der)
          
        # 현재 시간 및 현재 시간 서명값
        print("execution_time : " + execution_time)
        print("eformsign_signature : " + binascii.hexlify(signature).decode('utf-8'))


    .. code-tab:: php
        :title: PHP - keycheck.inc.php

        <?php
        namespace eformsignECDSA;
  
        class PublicKey
        {
          
            function __construct($str)
            {
                $pem_data = base64_encode(hex2bin($str));
                $offset = 0;
                $pem = "-----BEGIN PUBLIC KEY-----\n";
                while ($offset < strlen($pem_data)) {
                    $pem = $pem . substr($pem_data, $offset, 64) . "\n";
                    $offset = $offset + 64;
                }
                $pem = $pem . "-----END PUBLIC KEY-----\n";
                $this->openSslPublicKey = openssl_get_publickey($pem);
            }
        }

        class PrivateKey
        {         

            function __construct($str)
            {
                $pem_data = base64_encode(hex2bin($str));
                $offset = 0;
                $pem = "-----BEGIN EC PRIVATE KEY-----\n";
                while ($offset < strlen($pem_data)) {
                    $pem = $pem . substr($pem_data, $offset, 64) . "\n";
                    $offset = $offset + 64;
                }
                $pem = $pem . "-----END EC PRIVATE KEY-----\n";
                $this->openSslPrivateKey = openssl_get_privatekey($pem);
            }
        }


        function getNowMillisecond()
        {
          list($microtime,$timestamp) = explode(' ',microtime());
          $time = $timestamp.substr($microtime, 2, 3);
          
          return $time;
        }
         
         
        function Sign($message, $privateKey)
        {
            openssl_sign($message, $signature, $privateKey->openSslPrivateKey, OPENSSL_ALGO_SHA256);
            return $signature;
        }
        ?>

    .. code-tab:: php
        :title: PHP - test.php

        <?php
        require_once __DIR__ . '/keycheck.inc.php';
 
        use eformsignECDSA\PrivateKey;
         
         
        define('PRIVATE_KEY', '발급 받은 private key(String)');
         
         
        //private key 세팅
        $privateKey = new PrivateKey(PRIVATE_KEY);
         
         
        //execution_time - 서버 현재 시간
        $execution_time = eformsignECDSA\getNowMillisecond();
         
         
        //eformsign_signature 생성
        $signature = eformsignECDSA\Sign($execution_time, $privateKey);
         
         
        //현재 시간 및 현재 시간 서명값
        print 'execution_time : ' . $execution_time . PHP_EOL;
        print 'eformsign_signature : ' . bin2hex($signature) . PHP_EOL;
        ?>





API 제공 리스트
======================

eformsign API는 서명 생성을 위한 API와 문서 작성 및 처리를 위한 API, 멤버 및 그룹 관리 API로 구분되어 있습니다.



서명 생성을 위한 API
-------------------------

서명 생성을 위해 먼저 `Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\를 사용해 주십시오. 

``POST``: `Access Token 발급 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\


Access Token API에 대한 자세한 설명은 
`다음 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\ 에서 확인하실 수 있습니다.
 
.. caution:: 
   
   서명 생성에는 30초의 시간 제한이 있습니다. 30초 이내에 서명을 생성하고 토큰을 발급받아야 합니다. 
   또한, 서버 상의 시간과 현재 시간이 일치하지 않을 수 있습니다. Access Token API를 호출한 후 수신한 응답 메시지의 "execution_time"을 확인하세요.

   .. code:: JSON

      { "code": "4000002", "ErrorMessage": "The validation time has expired.",     "execution_time": 1611538409405 }

   `다음 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\ 의 예제 위치에도 "execution_time"을 입력하세요.
   
   |image5| 

   Access Token은 멤버 권한에 대해서도 발급받을 수 있습니다. 멤버에 대한 Access Token을 발급받으려면 다음과 같이 "execution_time" 외에 "member_id"를 함께 입력해 주세요. 
   
   |image6| 


   이후 해당 API를 실행하면 Access Token이 발급되며, 다음과 같은 형태의 응답을 수신할 수 있습니다.

   .. code:: JSON

      { "api_key": { "name": "애플리케이션_", "alias": "테스트용", "company": { "company_id": "dec5418e58694d90a65d6c38e3d226db", "name": "샘플 데모", "api_url": "https://kr-api.eformsign.com" } }, "oauth_token": { "expires_in": 3600, "token_type": "JWT", "refresh_token": "8fd0a3c1-44dc-4a03-96ad-01fa34cd159c", "access_token": "eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJlZm9ybXNpZ24uaWFtIiwiY29udGV4dCI6eyJjbGllbnRJZCI6IjY4MDk0ZWVhMjVhZjRhNjI5ZTI4ZGU5Y2ZlYzRlYmZjIiwiY2xpZW50S2V5IjoiZTNiM2IzZTUtMGEzMS00NTE1LWE5NzEtN2M4Y2FlNDI4NzZmIiwibWFuYWdlbWVudElkIjoiMzRhYWI4MDBjMmEwNDQwNThmZDRlZjc5OGFlY2RlY2EiLCJzY29wZXMiOiJzbWFydF9lZm9ybV9zY29wZSIsInR5cGUiOiJ1c2VyIiwidXNlck5hbWUiOiIzMmIzZDRmOC00MjdkLTRjZjQtOTZiYS1mYzAxNjIxNWRkNDciLCJ1c2VySWQiOiJhNTEyNGVkNmU2M2Y0OTMzOGJlOTA0MjVhNjFkYjlmNSIsInJlZnJlc2hUb2tlbiI6IjhmZDBhM2MxLTQ0ZGMtNGEwMy05NmFkLTAxZmEzNGNkMTU5YyJ9LCJjbGFpbSI6eyJjb21wYW55X2lkIjoiZGVjNTQxOGU1ODY5NGQ5MGE2NWQ2YzM4ZTNkMjI2ZGIiLCJhY2Nlc3Nfa2V5IjoiMzJiM2Q0ZjgtNDI3ZC00Y2Y0LTk2YmEtZmMwMTYyMTVkZDQ3In0sImV4cCI6MTYxMTU0MjIzNiwiaWF0IjoxNjExNTM4NjM2fQ.BltoXXBSabjXfpyLsZik9OZTE5XtLqe9lguMmJ_qfwZN1NyoVoxDqA5y1-_TLis7FvvNjfI1eegOroCZDZPFyXRaBxAj0CW8TijVjbhliJBuccHFyKXaJxmo_GMmTHYtxNNB1SUgLeFIrYROnpQndU8J7ZkfPDgYGwh1YSx-5s4" } }





.. caution:: 
   
   발급한 API 키는 `다음 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\ 의 위치에 있는 **Authorize** 버튼(|image4|)을 눌러 등록하세요. 단, API 키 값에는 **반드시 Base64**\ 로 인코딩한 문자열을 넣어야 합니다. https://www.base64encode.org/ 에 접속하여 발급한 API 키를 넣고 인코딩된 텍스트를 받아 삽입하세요. 


.. note:: 
  
   Access Token API의 **Authorize** 버튼에는 API 키 값을 입력해야 합니다.


------------


문서 작성 및 처리를 위한 API
----------------------------------

서명을 생성한 후에는 다음의 문서 API를 사용해 새 문서를 작성하거나 문서 정보를 조회할 수 있으며, 완료 문서 파일(문서 PDF, 감사추적증명서) 다운로드하거나 문서의 첨부파일을 다운로드할 수 있습니다. 


.. caution:: 

   본 문서 API를 사용하기 위해서는 Access Token을 먼저 발급받아야 합니다. `Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\으로 발급받은 Access Token을 `다음 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/>`__\ 의 위치에 있는 **Authorize** 버튼(|image4|)을 눌러 등록하세요. 


.. note:: 
  
   문서 API의 **Authorize** 버튼에는 Access Token 값을 입력해야 합니다. 


현재 제공 중인 `문서 API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/document>`_\는 다음과 같습니다.



``POST``: `새 문서 작성_내부 수신자 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents>`_\ 

``POST``: `새 문서 작성_외부 수신자 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/post-api-documents-external>`_\ 

``GET``: `문서정보 조회 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID>`_\

``GET``: `문서 파일 다운로드_문서 PDF 및 감사추적증명서 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_files>`_\

``GET``: `문서 첨부 파일 다운로드 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_attach_files>`_\ 

``GET``: `문서 목록 조회 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-documents>`_\ 

``DELETE``: `문서 삭제 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-documents>`_\ 

``POST``: `외부 수신자 문서 재요청 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents-document_id-re_request_outsider>`_\ 

``GET``: `작성가능한 템플릿 목록 조회 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-forms>`_\  

``POST``: `문서 일괄 작성 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

-----------------


멤버 및 그룹 관리를 위한 API
----------------------------------

API를 사용해 멤버 및 그룹을 관리할 수 있습니다. 멤버 및 그룹 목록을 조회할 수 있으며, 멤버 및 그룹을 수정, 삭제할 수 있습니다. 


.. caution:: 

   본 API를 사용하기 위해서는 Access Token을 먼저 발급받아야 합니다. `Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0_auth>`_\으로 발급받은 Access Token을 `다음 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/>`__\ 의 위치에 있는 **Authorize** 버튼(|image4|)을 눌러 등록하세요. 


.. note:: 
  
   API의 **Authorize** 버튼에는 Access Token 값을 입력해야 합니다. 


현재 제공 중인 `멤버 및 그룹 관리 API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/members>`_\는 다음과 같습니다.


멤버 관리 API
^^^^^^^^^^^^^^^^^


``GET``: `멤버 목록 조회 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-members>`_\   

``PATCH``: `멤버 수정 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-members-member_id>`_\  

``DELETE``: `멤버 삭제 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-members-member_id>`_\  


그룹 관리 API
^^^^^^^^^^^^^^^^^


``GET``: `그룹 목록 조회 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-groups>`_\  

``POST``: `그룹 추가 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-groups>`_\  

``PATCH``: `그룹 수정 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-groups>`_\  

``DELETE``: `그룹 삭제 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-groups>`_\  



.. note:: 

    각 eformsign API에 대한 자세한 설명은 `다음 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/>`__\ 에서 확인하실 수 있습니다.






API 코드
===================

Open API 사용 시 다음의 코드를 참조하시기 바랍니다.


API 상태 코드
--------------

API 상태 코드는 다음과 같습니다.

200
^^^^^^^^

===========  ===============  ===================================
Code         설명              비고
===========  ===============  ===================================
200          성공              성공
===========  ===============  ===================================


202
^^^^^^^^

===========  ===============  ===================================
Code         설명              비고
===========  ===============  ===================================
2020001      PDF 생성 중        - PDF 파일 다운로드 시 파일은 비동기로 생성되기 때문에 문서 저장 후 PDF 생성까지 시간이 추가로 필요함
                                - 수초에서 수분 내에 재요청시 다운로드 가능
===========  ===============  ===================================


400
^^^^^^^^

===========  =======================  ==========================================
Code         설명                      비고
===========  =======================  ==========================================
4000001      필수 입력값 누락           API의 필수 입력값(헤더 값 또는 파라미터)이 누락됐을 경우                        
4000002      인증 시간 만료             API 인증 요청 시간이 만료되었을 경우
4000003      API 키가 존재하지 않음     API 키가 삭제되었거나 잘못 입력했을 경우
4000004      문서가 존재하지 않음       문서 ID를 잘못 입력했을 경우
4000005      회사가 존재하지 않음       회사가 삭제되었을 경우
===========  =======================  ==========================================


403
^^^^^^^^

===========  =========================  ==========================================
Code         설명                        비고
===========  =========================  ==========================================
4030001      접근 권한 없음              API가 비활성화 되었을 경우
4030002      access_token 인증 오류      Access Token이 올바르지 않을 경우
4030003      refresh_token 인증 오류     Refresh Token이 올바르지 않을 경우
4030004      서명값 검증 실패            서명값이 올바르지 않을 경우
4030005      지원하지 않는 API           지원하지 않는 API 호출 시
===========  =========================  ==========================================

405
^^^^^^^^

===========  =====================  ===================================
Code         설명                    비고
===========  =====================  ===================================
4050001      지원하지 않는 method     지원하지 않는 method 호출 시
===========  =====================  ===================================


500
^^^^^^^^

===================  ===============  ===================================
Code                 설명              비고
===================  ===============  ===================================
5000001~5000003      서버 에러         서버에 오류가 발생했을 경우
===================  ===============  ===================================


---------


Step 타입
--------------

===========  ===============  ===================================
Type         Code             설명
===========  ===============  ===================================
Start         00               시작 단계
Complete      01               완료 단계
Approval      02               결재 단계
External      03               외부수신인 단계
Accept        04               내부수신인 단계
Participant   05               참여자
Reviewer      06               검토자
===========  ===============  ===================================


User 타입
--------------

===========  ===============  ===================================
Type         Code             설명
===========  ===============  ===================================
내부 멤버     01                내부 멤버인지 여부
외부 사용자   02                내부 멤버가 아닌 외부 사용자 여부
===========  ===============  ===================================

Status 타입
--------------

===========  ===============  ==================================================
Type          Code             설명
===========  ===============  ==================================================
초안          00                시작단계 임시저장된 문서 상태
진행 중       01                결재 요청, 외부자 요청, 열람, 내부자 요청 단계 상태
수정 중       02                문서를 수정 중일 때 (멤버, 최초 작성자)
완료          03                문서가 완료되었을 때
반려          04                결재 반려, 내부자 반려, 외부자 반려
취소          05                문서가 취소된 상태
취소 요청     06                내부 수신인 단계
===========  ===============  ==================================================



Action 타입
--------------

==========================  ===============  ===================================
Type                         Code             설명
==========================  ===============  ===================================
doc_tempsave                 001              문서 임시 저장
doc_create                   002              문서 생성
doc_complete                 003              문서 최종 완료
doc_request_approval         010              결재 요청
doc_reject_approval          011              결재 반려
doc_accept_approval          012              결재 승인
doc_cancel                   013              결재 요청 취소
doc_request_reception        020              내부 멤버 요청
doc_reject_reception         021              내부 멤버 반려
doc_accept_reception         022              내부 멤버 승인
doc_accept_tempsave          023              내부 멤버 임시 저장
doc_request_outsider         030              외부수신자 요청
doc_reject_outsider          031              외부수신자 반려
doc_accept_outsider          032              외부수신자 승인
doc_rerequest_outsider       033              외부수신자 재요청
doc_open_outsider            034              외부수신자 열람
doc_outsider_tempsave        035              외부수신자 임시 저장
doc_request_revoke           040              문서 취소 요청
doc_refuse_revoke            041              문서 취소 요청 거절
doc_revoke                   042              문서 취소
doc_update                   043              문서 수정
doc_cancel_update            044              문서 수정 취소
doc_request_reject           045              문서 반려 요청
doc_refuse_reject            046              문서 반려 요청 거절
doc_request_delete           047              문서 삭제 요청
doc_refuse_delete            048              문서 삭제 요청 거절
doc_delete                   049              문서 삭제
doc_complete_send_pdf        050              완료 문서 PDF 전송
doc_transfer                 051              문서 이관
doc_request_participant      060              참여자 요청
doc_reject_participant       061              참여자 반려
doc_accept_participant       062              참여자 승인
doc_rerequest_participant    063              참여자 재요청(외부자)
doc_open_participant         064              참여자 문서 열람(외부자)
doc_request_reviewer         070              검토자 요청
doc_reject_reviewer          071              검토자 반려
doc_request_reviewer         072              검토자 승인 
doc_rerequest_reviewer       073              검토자 재요청(외부자)
doc_open_review              074              검토자 문서 열람(외부자)
==========================  ===============  ===================================




상세 문서 상태 타입
-----------------------

==========================  ===============  ===================================
Type                         Code             설명
==========================  ===============  ===================================
doc_tempsave                 001              초안 (최초 작성자 임시저장 상태)
doc_create                   002              문서 작성
doc_complete                 003              문서 완료
doc_update                   043              문서 수정
doc_request_delete           047              문서 삭제 요청
doc_delete                   049              문서 삭제
doc_request_revoke           040              문서 취소 요청
doc_revoke                   042              문서 취소
doc_request_reject           045              문서 반려 요청
doc_request_approval         010              결재 요청
doc_accept_approval          012              결재 승인
doc_reject_approval          011              결재 반려
doc_cancel                   013              결재 취소
doc_request_reception        020              내부 멤버 요청
doc_accept_reception         022              내부 멤버 승인
doc_reject_reception         021              내부 멤버 반려
doc_request_outsider         030              외부수신자 요청
doc_accept_outsider          032              외부수신자 승인
doc_reject_outsider          031              외부수신자 반려
doc_request_participant      060              참여자 요청
doc_accept_participant       062              참여자 승인
doc_reject_participant       061              참여자 반려
doc_request_reviewer         070              검토자 요청
doc_accept_reviewer          072              검토자 승인 
doc_reject_reviewer          071              검토자 반려
==========================  ===============  ===================================






.. |image1| image:: resources/company_id.png
   :width: 600px
.. |image2| image:: resources/column_icon.png
.. |image3| image:: resources/document_id.png
.. |image4| image:: resources/authorize_icon.png
.. |image5| image:: resources/execution_time.png
   :width: 450px
.. |image6| image:: resources/execution_time2.png
   :width: 450px