----------------------------
eformsign Webhook 사용하기
----------------------------

eformsign에 이벤트가 발생했을 때 발생한 이벤트의 정보를 고객 시스템/서비스로 알려주는 기능입니다. Webhook을 설정하면, 고객의 Webhook endpoint로 해당 이벤트 정보를 HTTP POST형식으로 알려 줍니다.

.. tip:: 

   Webhook의 endpoint는 고객의 client callback URL을 뜻합니다. Open API를 지속적으로 호출하여 변경사항을 체크하는 방법(polling)과 비교하여 불필요한 호출 없이 eformsign 상의 이벤트에 대한 정보를 얻을 수 있습니다.


시작하기 
=========


.. _webhook:

Webhook 키 발급하기
--------------------

1. eformsign에 대표 관리자로 로그인 후, 메뉴 트리에서 **[커넥트] > [API / Webhook]** 페이지로 이동합니다. 

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: 커넥트 > API/Webhook 메뉴 위치


2. **[Webhook 관리]** 탭을 선택하고 **Webhook 추가** 버튼을 클릭합니다.

.. image:: resources/webhook2.PNG
    :width: 700
    :alt: Webhook 추가 버튼


3. Webhook 키 생성 팝업창에 이름, Webhook URL, 활성 상태, 적용 템플릿을 입력하고 등록 버튼을 클릭합니다.

.. image:: resources/webhook3.PNG
    :width: 700
    :alt: Webhook 키 생성 팝업창


4. 생성된 Webhook 목록에서 **키보기** 버튼을 클릭하여 Webhook 공개키를 확인합니다.

.. image:: resources/webhook4.PNG
    :width: 700
    :alt: Webhook 키보기 버튼 위치

.. image:: resources/webhook5.PNG
    :width: 700
    :alt: Webhook 키 확인 



.. caution:: 

    **키 재발행** 버튼을 클릭하면 해당 Webhook의 공개 키가 재발행되며, 이전의 키는 사용할 수 없게 됩니다.

.. note:: **Webhook 정보 수정 방법**

    생성된 Webhook 목록에서 **수정** 버튼을 클릭하여 Webhook 정보를 변경할 수 있습니다.


.. note:: **Webhook 삭제 방법**

    생성된 Webhook 목록에서 **삭제** 버튼을 클릭하여 Webhook을 삭제할 수 있습니다.    



5. 생성된 Webhook 리스트에서 **테스트** 버튼을 클릭하면 테스트 Webhook을 전송하고 결과를 반환합니다.

.. image:: resources/webhook6.PNG
    :width: 700
    :alt: Webhook 테스트 확인 

다음은 테스트를 위한 json 파일입니다.

.. code:: JSON

	{
	"webhook_id" : "해당 Webhook ID",
	"webhook_name" : "해당 Webhook 이름",
	"company_id" : "회사의 ID",
	"event_type" : “document”,
	"document" : {
	  "id" : “test_doc_id”,
	   "template_id" : “test_template_id”,
	   "template_version" : “1”,
	   "document_history_id" : “test_document_history_id”,
	   "doc_status" : “doc_create”,
	   "editor_id" : "사용자 ID",
	   "updated_date" : "현재 시간(UTC Long)"
	}
	}
	Test URL : 해당 Webhook의 URL




서명 생성하기 
==============


서명 생성 방법에 대해서는  Java, Python, PHP 언어별로 설명합니다.

Java
-------

eformsign 서버로부터 전달받은 이벤트 정보를 `Webhook Key 발급하기 <#webhook>`__\에서 발급받은 public key로 검증하여 eformsign에서 정상적으로 호출한 이벤트인지에 대한 검증을 진행합니다. 

.. note:: 

  서명 알고리즘은 SHA256withECDSA을 사용합니다.


Python
-------

키 포맷 처리용 라이브러리를 사용해야 합니다. 작업전 다음의 명령어를 통해 해당 라이브러리를 설치하세요.

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

다음 예제의 keycheck.inc.php, test.php 파일이 동일한 패스에 위치하게 한 후에 진행해야 합니다.


각 언어별 예제
---------------------

다음은 각 언어별 예제입니다.

.. code-tabs::

    .. code-tab:: java
        :title: java

        import java.io.*;
        import java.math.BigInteger;
        import java.security.*;
        import java.security.spec.X509EncodedKeySpec;
         
        ....
        /**
         *  request에서 header와 body를 읽습니다.
         *
         */
         
         
        //1. get eformsign signature
        //eformsignSignature는 request header에 담겨 있습니다.
        String eformsignSignature = request.getHeader("eformsign_signature");
         
         
        //2. get request body data
        // eformsign signature 검증을 위해 body의 데이터를 String으로 변환 합니다.
        String eformsignEventBody = null;
        StringBuilder stringBuilder = new StringBuilder();
        BufferedReader bufferedReader = null;
         
        try {
            InputStream inputStream = request.getInputStream();
            if (inputStream != null) {
                bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                char[] charBuffer = new char[128];
                int bytesRead = -1;
                while ((bytesRead = bufferedReader.read(charBuffer)) > 0) {
                    stringBuilder.append(charBuffer, 0, bytesRead);
                }
            }
         } catch (IOException ex) {
            throw ex;
         } finally {
            if (bufferedReader != null) {
                try {
                    bufferedReader.close();
                } catch (IOException ex) {
                    throw ex;
                }
            }
         }
        eformsignEventBody = stringBuilder.toString();
         
         
         
         
        //3. publicKey 세팅
        String publicKeyHex = "발급 받은 Public Key(String)";
        KeyFactory publicKeyFact = KeyFactory.getInstance("EC");
        X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(new BigInteger(publicKeyHex,16).toByteArray());
        PublicKey publicKey = publicKeyFact.generatePublic(x509KeySpec);
         
        //4. verify
        Signature signature = Signature.getInstance("SHA256withECDSA");
        signature.initVerify(publicKey);
        signature.update(eformsignEventBody.getBytes("UTF-8"));
        if(signature.verify(new BigInteger(eformsignSignature,16).toByteArray())){
            //verify success
            System.out.println("verify success");
            /*
             * 이곳에서 이벤트에 맞는 처리를 진행합니다.
             */
        }else{
            //verify fail
            System.out.println("verify fail");
        }


    .. code-tab:: python
        :title: Python 3.9.6

        import hashlib
    		import binascii
    		 
    		from ecdsa import VerifyingKey, BadSignatureError
    		from ecdsa.util import sigencode_der, sigdecode_der
    		from flask import request
    		 
    		 
    		...
    		# request에서 header와 body를 읽습니다.
    		# 1. get eformsign signature
    		# eformsignSignature는 request header에 담겨 있습니다.
                eformsign_signature_str= request.headers['eformsign_signature']
                eformsign_signature = binascii.unhexlify(eformsign_signature_str)
    		 
    		 
    		# 2. get request body data
    		# eformsign signature 검증을 위해 body의 데이터를 String으로 변환합니다.
    		data = request.data
    		 
    		 
    		# 3. publicKey 세팅
    		publicKeyHex = "발급받은 public key"
    		publickey = VerifyingKey.from_der(binascii.unhexlify(publicKeyHex))
    		 
    		 
    		# 4. verify
    		try:
    		    if publickey.verify(eformsign_signature, data, hashfunc=hashlib.sha256, sigdecode=sigdecode_der):
    		        print("verify success")
    		        # 이곳에 이벤트에 맞는 처리를 진행합니다.
    		except BadSignatureError:
    		    print("verify fail")

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
         
        function Verify($message, $signature, $publicKey)
        {
            return openssl_verify($message, $signature, $publicKey->openSslPublicKey, OPENSSL_ALGO_SHA256);
        }
        ?>

    .. code-tab:: php
        :title: PHP - test.php

        <?php
        require_once __DIR__ . '/keycheck.inc.php';
        use eformsignECDSA\PublicKey;
         
        define('PUBLIC_KEY', '발급 받은 public key를 넣어주세요.');
        ...
        /*
         *  request에서 header와 body를 읽습니다.
         *
         */
         
         
        //1. get eformsign signature
        //eformsignSignature는 request header에 담겨 있습니다.
        $eformsignSignature = $_SERVER['HTTP_eformsign_signature'];
         
         
        //2. get request body data
        // eformsign signature 검증을 위해 body의 데이터를 읽습니다.
        $eformsignEventBody = json_decode(file_get_contents('php://input'), true);
         
         
        //3. publicKey 세팅
        $publicKey = new PublicKey(PUBLIC_KEY);
         
         
        //4. verify
        $ret = - 1;
        $ret = eformsignECDSA\Verify(MESSAGE, $eformsignSignature, $publicKey);
          
        if ($ret == 1) {
            print 'verify success' . PHP_EOL;
            /*
             * 이곳에서 이벤트에 맞는 처리를 진행합니다.
             */
        } else {
            print 'verify fail' . PHP_EOL;
        }
         ...
          
        ?>




Webhook 제공 리스트
====================

다음의 Webhook을 설정하면 해당 이벤트 발생 시 설정한 Webhook endpoint로 변경 정보를 수신할 수 있습니다. 

현재 제공 중인 `Webhook <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`_\은 다음과 같습니다.


``POST``: `/webhook document event <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook#/default/post-webhook-document-event>`_\  웹훅 전송


각 eformsign Webhook에 대한 자세한 설명은 
`다음 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`__\ 에서 확인하실 수 있습니다.




Webhook 관련 정보
===================

eformsign은 Webhook 이벤트로 **문서** 이벤트와 **PDF 생성** 이벤트를 제공하고 있습니다.


문서 이벤트
-------------

eformsign에서 문서의 생성 또는 상태 변경 시 발생하는 이벤트입니다.


.. table:: 

   ================ ====== ================
   Name             Type   설명
   ================ ====== ================
   id               String 문서 ID
   template_id      String 템플릿 ID
   template_name    String 템플릿 명
   template_version String 템플릿 제목
   workflow_seq     int    워크플로우 순서
   workflow_name    String 워크플로우 명칭
   history_id       String 문서 히스토리 ID
   status           String 문서 상태
   editor_id        String 작성자 ID
   updated_date     long   문서 변경시간
   ================ ====== ================


이벤트 데이터 중 문서 상태를 나타내는 status의 의미는 다음을 참조하세요.

.. _status: 

.. table:: 

   ========================== ==================
   Name                       설명
   ========================== ==================
   doc_create                 문서 생성시
   doc_tempsave               문서 임시 저장시
   doc_request_approval       결재 요청시
   doc_accept_approval        결재 승인시
   doc_reject_approval        결재 반려시
   doc_request_external       외부자 요청시
   doc_remind_external        외부자 재 요청시
   doc_open_external          외부자 열람시
   doc_accept_external        외부자 승인시
   doc_reject_external        외부자 반려시
   doc_request_internal       내부자 요청시
   doc_accept_internal        내부자 승인시
   doc_reject_internal        내부자 반려시
   doc_tempsave_internal      내부자 임시 저장시
   doc_cancel_request         요청 취소시
   doc_reject_request         반려 요청시
   doc_decline_cancel_request 반려 요청 거절시
   doc_delete_request         삭제 요청시
   doc_decline_delete_request 삭제 요청 거절시
   doc_deleted                문서 삭제시
   doc_complete               문서 완료시
   ========================== ==================


PDF 생성 이벤트
----------------

eformsign에서 문서의 PDF 파일이 생성될 때 발생하는 이벤트입니다.

.. table:: 

   ===================== ====== ================
   Name                  Type   설명
   ===================== ====== ================
   document_id           String 문서 ID
   template_id           String 템플릿 ID
   template_name         String 템플릿 명
   template_version      String 템플릿 제목
   workflow_seq          int    워크플로우 순서
   workflow_name         String 워크플로우 명칭
   document_history_id   String 문서 히스토리 ID
   document_status       String 문서 상태
   ===================== ====== ================


이벤트 데이터 중 문서 상태를 나타내는 status의 의미는 `다음 <#status>`__\을 참조하세요.

