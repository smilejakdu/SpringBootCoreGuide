# JWT

## JWT

JWT 는 점 으로 구분된 아래의 세 부분으로 구성된다.

* 헤더 (Header) : 토큰의 타입(JWT) 과 사용된 알고리즘(HMAC, SHA256 또는 RSA 등) 을 정의한다. 보통 JSON 객체로 표현되며, 이 객체는 Base64Url 로 인코딩 됩니다.
* 내용 (Payload): 토큰에 포함될 클레임을 담고있습니다. 클레임은 엔티티 및 추가 데이터에 대한 진술 또는 주장입니다. 페이로드도 Base64Url 로 인코딩되어 JWT 의 두번째 부분을 형성합니다.
* 서명 (Signature) : 헤더의 인코딩된 값, 페이로드의 인코딩된 값, 비밀키를 사용하여 생성됩니다. 이 서명은 토큰이 중간에 변경되지 않았음을 보장합니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-04 오후 7.48.08.png" alt=""><figcaption></figcaption></figure>

alg 속성에서는 해싱 알고리즘을 지정합니다. 해싱 알고리즘은 보통 SHA256 또는 RSA를 사용하며, 토큰을 검증할 때 사용되는 서명 부분에서 사용됩니다.

## 내용

JWT 의 내용에는 토큰에 담는 정보를 포함합니다.

이곳에 포함된 속성들은 클레임이라 하며, 크게 세가지로 분류됩니다.

* 등록된 클레임
* 공개 클레임
* 비공개 클레임

등록된 클레임은 필수는 아니지만 토큰에 대한 정보를 담기 위해 이미 이름이 정해져 있는 클레임을 뜻한다.

* iss: JWT 의 발급자 주체를 나타낸다. iss 의 값은 문자열이나 URI를 포함하는 대소문자를 구분하는 문자열이다.
* sub: JWT 의 제목입니다.
* aud: JWT 의 수신인입니다. JWT를 처리하려는 각 주체는 해당 값으로 자신을 식별해야합니다. 요청을 처리하는 주체가 aud 값으로 자신을 식별하지 않으면 JWT는 거부 됩니다.
* exp: JWT 의 만료시간입니다. 시간은 NumericDate 형식으로 지정해야 합니다.
* nbf: Not Before 를 의미합니다.
* iat : JWT 가 발급된 시간입니다.
* jti: JWT 의 식별자 입니다. 주로 중복 처리를 방지하기 위해 사용됩니다.

공개 클레임은 키 값을 마음대로 정의할 수 있습니다. 다만 충돌이 발생하지 않을 이름으로 설정해야 합니다.

비공개 클레임은 통신 간에 상호 합의되고 등록된 클레임과 공개된 클레임이 아닌 클레임을 의미합니다.

```markdown
{
    "sub": "wikibooks payload",
    "exp": "160123123",
    "userId": "wikibooks",
    "username": "flature"
}
```

완성된 내용은 Base64Url 형식으로 인코딩 되어 사용됩니다.



## 서명

JWT 의 서명 부분은 인코딩된 헤더, 인코딩된 내용, 비밀키, 헤더의 알고리즘 속성값을 가져와 생성됩니다.

서명은 토큰의 값들을 포함해서 암호화하기 때문에 메시지가 도중에 변경되지 않았는지 확인할 때 사용됩니다.


