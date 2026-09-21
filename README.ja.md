<h1  align="center">
    <img src="https://fiware.github.io/tutorials.Step-by-Step/img/fiware-farm.png" />
    <img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"/>
    <br/>
    👨‍🌾 👩‍🌾 🐄 🐐 🐑 🐖 🐓 🌻 🥕 🌽
</h1>

## Verifiable Credentials を理解する

[![FIWARE Security](https://fiware.github.io/catalogue/badges/chapters/security.svg)](https://github.com/FIWARE/catalogue/blob/master/security/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Understanding-At-Context.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)

このチュートリアルでは、Verifiable Credential (検証可能な資格情報) と Decentralised Identifier (分散型識別子) の概念、
およびそれらを Data Space にどのように適用するかを紹介します。発行者 (issuer) が受領者のために資格情報をどのように
生成するか、そして資格情報の保有者 (holder) がその主張を裏付ける Verifiable Presentation をどのように発行できるかを
説明します。これらの実践的な例は、後続のチュートリアルで定義される Data Space Connector のさまざまなコンポーネントの
役割を理解する助けとなります。

このチュートリアルでは、GUI を使用した操作例に加えて、REST API を使用して資格情報を生成するための
[cUrl](https://ec.haxx.se/) コマンドの例も示します。

## コンテンツ

<details>
<summary><strong>Details</strong></summary>

</details>

# Verifiable Credentials

> **Reagan:** “But the importance of this treaty transcends numbers. We have listened to the wisdom in an old Russian
> maxim. And I'm sure you're familiar with it, Mr. General Secretary, though my pronunciation may give you difficulty.
> The maxim is: доверяй, но проверяй - trust, but verify.”
>
> **Gorbachev:** “You repeat that at every meeting.“
>
> **Reagan:** “I like it.”
>
> ― Remarks on Signing the Intermediate-Range Nuclear Forces Treaty

## Verifiable Credentials とは？

Verifiable Credential (検証可能な資格情報) は、会員証や運転免許証のようなもののデジタル版です。ユーザが保持している
と主張する、何らかの所有権や権利を表すものです。Verifiable Credential は国際的な W3C 標準に準拠しており、暗号学的に
安全であるため、現実世界と同じように検証することができます。

Verifiable Credential の背景にある考え方は、誰でもそれを発行 (issue) し検証 (verify) できるということです。つまり、
すべての情報を管理する中央集権的な所有者は存在しません。これは、ユーザがどこかにログインして自分が誰であるかを証明
することに依存する、標準的な OAuth2 Authorization Code Grant フローとは対照的です。

たとえば、現実の世界では、観光客が新しい国に入国するとき、通常は入国審査を通過する必要があり、そこで有効な身分証明書
またはパスポートの提示を求められます。国境警備官は、パスポートが本物であることと、パスポートが実際にその観光客自身の
ものであることの両方を確認する必要があります。特定のビザや予防接種証明書など、他の要件を満たす必要がある場合もあり
ます。

さて、観光客のパスポートは彼ら自身の本国政府によって発行されているため、国境警備官は、写真が本人と一致することを
確認するのと同時に、第三者によって提供された文書が本物であることを、必ずしも当該国に直接問い合わせることなく、暗黙
のうちに確認していることになります。

Verifiable Credential では、何らかの合意された暗号学的証明に基づいて、文書の有効性のチェックを行うことができます。
デコードされた文書は主張された権利を保持しており、さらに資格情報のサブジェクト (パスポートの写真に相当) と発行者の
身元 (パスポート自体の発行国に相当) の両方に直接結びついています。いずれの場合も、この身元は一意の ID に解決される
必要があり、その ID は所有者によってあらかじめ生成されています。

## Decentralised Identifiers とは？

[Decentralised Identifier](https://www.w3.org/TR/did-1.0/) (分散型識別子, DID) は、中央機関に頼ることなく、検証可能
で永続的な識別子を作成するための仕組みです。デジタル識別子は、コロンで区切られたいくつかのセクションから構成される
URN です。まず名前空間 `did` から始まり、続いて分散型識別子のメソッド (`web` や `ethr`、`key` など) が続きます。
このメソッドは、識別子の残りの部分をどのようにデコードして解決するかを定義します。たとえば
[`did:web`](https://w3c-ccg.github.io/did-method-web/) という用語は、公開されている Web ドメイン上でホストされる
分散型識別子を作成するためのメソッドを指し、`did:ethr` は
[Etherium ベースの ID](https://github.com/uport-project/ethr-did-registry) で使用され、
[`did:key`](https://w3c-ccg.github.io/did-key-spec/) はエンコードされた公開鍵を保持します。URN の残りのセクションは、
検証者 (verifier) が ID が正しく使用されているかどうかを確認できるように解決されます。

たとえば `did:web:fiware.github.io:tutorials.Step-by-Step:alice` は、
[`https://fiware.github.io/tutorials.Step-by-Step/alice/did.json`](https://fiware.github.io/tutorials.Step-by-Step/alice/did.json)
にある文書を参照しています。

```json
{
    "@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/suites/jws-2020/v1"],
    "id": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
    "verificationMethod": [
        {
            "id": "did:fiware.github.io:tutorials.Step-by-Step:alice#owner",
            "type": "JsonWebKey2020",
            "controller": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
            "publicKeyJwk": {
                "kty": "EC",
                "crv": "secp256k1",
                "x": "Nd3DeQ7G/1pTeYM6viWK6plbSD9E7cA9C2ONG9qG3CQ=",
                "y": "LuMt0dFWni1/fs/VqfjNOHAZT3PWGxKU8kUlLffGtjM="
            }
        }
    ],
    "authentication": ["did:web:fiware.github.io:tutorials.Step-by-Step:alice#owner"],
    "assertionMethod": ["did:web:fiware.github.io:tutorials.Step-by-Step:alice#owner"]
}
```

これは、`id` を検証するために使用できる検証メソッドのリストを保持する JSON-LD 文書です。この場合、`JsonWebKey2020`
は [JSON Web Signature](https://w3c-ccg.github.io/lds-jws2020/) を指します。

# アーキテクチャ

このチュートリアルの目的のために、前のチュートリアルのデモ用 Farm Management Information System (FMIS) を使用し、
獣医 (vet) の Context Broker へのアクセスを変更します。私たちの Data Space 内には、Farmer、Vet、Contract labourer が
それぞれ所有する、実質的に3つの Context Broker があることを思い出してください。

-   **Building** データを保持し、すべてのシステムからのデータを照合するために使用されるデフォルト・テナント
-   **Animal**、**Device**、**AgriParcel** 情報を保持する `farmer` テナント
-   追加のケアが必要な動物に関する **Animal** データを保持する `contractor` テナント
-   新生児の動物に関する **Animal** データを保持する `vet` テナント

Data Space 内で、**Vet** は自分のデータへのアクセスを正当なユーザのみに制限したいと考えています :

-   彼女のデータへのアクセスを購入した、**Vets Mart** の認定ユーザ
-   国の **Government** によって認定された、法律上アクセスが許可されている動物福祉担当官

このチュートリアルでは、アクセス・ルールの強制を完成させることはしませんが、認定されたユーザが実際にそれらの組織の
一員であり、特定のロールを保持していることをどのように証明できるかを示します。

したがって、全体的なアーキテクチャは次の要素で構成されます :

-   [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してリクエストを送受信します。これは、それぞれ独自のテナントで実行される以下のシステムに分割されています :
    -   デフォルト・テナント
    -   `farmer` テナント
    -   `contractor` テナント
    -   `vet` テナント
-   FIWARE [IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/) は
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してサウスバウンド・リクエストを受信し、それらをデバイスのための
    [JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    コマンドに変換します
-   基盤となる [MongoDB](https://www.mongodb.com/) データベース :
    -   **Orion Context Broker** が、データ・エンティティ、サブスクリプション、レジストレーションなどのコンテキスト・
        データ情報を保持するために使用します
    -   **IoT Agent** が、デバイスの URL やキーなどのデバイス情報を保持するために使用します
-   システム内のコンテキスト・エンティティを定義する静的な `@context` ファイルを提供する HTTP **Web-Server**
-   **チュートリアル・アプリケーション**は、以下のことを行います :
    -   HTTP を介して実行される
        [JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        プロトコルを使用して、ダミーの[農業 IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)
        のセットとして機能します
    -   稼働中の Farm Management Information System (FMIS) を表示します

さらに、私たちの **Vet** はダミーの Data Space Connector によって保護されており、この Data Space Connector は
[VC Verifier](https://github.com/FIWARE/VCVerifier) の役割を模擬しています。これは、さらに他の FIWARE Data Space
コンポーネントに接続します :

-   信頼すべきサービスの場所を保持する FIWARE
    [Credentials Configuration Service](https://github.com/FIWARE/credentials-config-service)
-   各発行者の資格情報について、信頼されたロールのリストを返す FIWARE
    [Trusted Issuers List](https://github.com/FIWARE/trusted-issuers-list/)

![](https://fiware.github.io/tutorials.Verifiable-Credentials/img/architecture.png)

要素間のすべての相互作用は HTTP リクエストによって開始されるため、エンティティをコンテナ化して、公開されたポートから
実行できます。

# 起動

すべてのサービスは、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.Verifiable-Credentials/blob/NGSI-LD/services) Bash スクリプトを実行する
ことによって、コマンドラインから初期化できます。以下のコマンドを実行して、リポジトリのクローンを作成し、必要な
イメージを作成してください :

```console
git clone https://github.com/FIWARE/tutorials.Verifiable-Credentials.git
cd tutorials.Verifiable-Credentials
git checkout NGSI-LD

./services orion|scorpio|stellio
```

> [!NOTE]
>
> クリーンアップしてやり直す場合は、次のコマンドを実行してください :
>
> ```
> ./services stop
> ```

# Verifiable Credentials

プレーン・テキストでデコードされた Verifiable Credential は、どんなものでも主張できてしまいます。Verifiable
Credential は通常、`type: VerifiableCredential` を持つ JSON-LD の断片です - 以下の例では、**Animal Welfare** 部門が
**Alice** に **Data Access Claim** を発行したいと考えています。**Data Access** のロールの詳細が主張 (claim) です。
**Animal Welfare** が資格情報を作成しているため、彼らが発行者 (issuer) であり、**Alice** がサブジェクトです。発行者
は自分の秘密鍵で資格情報に署名します。

資格情報の署名に使用される秘密鍵は共有すべきではありませんが、このチュートリアルでは、すべてのユーザのすべての
リクエストを通じて `0b6366519a40eb4f384f7f84cf8bb716683ad1af8adbe60e59fe24ba042e396a` が使用されます。これは、対応する
公開鍵が [分散型識別子](https://fiware.github.io//tutorials.Step-by-Step/alice/did.json) として公開 Web 上に保存され
ているためです。必要な情報は、以下に示すスクリプトを使用して生成できます。

```javascript
import crypto from "crypto";
import elliptic from "elliptic";

// Request a 32 byte key
const size = parseInt(process.argv.slice(2)[0]) || 32;
const randomString = crypto.randomBytes(size).toString("hex");
const key = randomString;

console.log(`Key (hex): ${key}`); // 0b6366519a40eb4f384 etc.

// Calculate the `secp256k1` curve and build the public key
const ec = new elliptic.ec("secp256k1");
const prv = ec.keyFromPrivate(key, "hex");
const pub = prv.getPublic();
console.log(`Public (hex): ${prv.getPublic("hex")}`);
console.log(`x (hex): ${pub.x.toBuffer().toString("hex")}`);
console.log(`y (hex): ${pub.y.toBuffer().toString("hex")}`);
console.log(`x (base64): ${pub.x.toBuffer().toString("base64")}`);
console.log(`y (base64): ${pub.y.toBuffer().toString("base64")}`);
console.log(`-- kty: EC, crv: secp256k1`);
```

## Verifiable Credential の生成

**Alice** は **Animal Welfare** 部門で働いているため、**Vet** の Context Broker にアクセスするには、そこで働いて
いることを証明する Verifiable Credential を持つ必要があります。

そのため **Animal Welfare** 部門 `did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare` は、**Alice**
`did:web:fiware.github.io:tutorials.Step-by-Step:alice` に **Access Claim** を含む Verifiable Credential を発行し
ます。これは秘密鍵を使って署名できます。固定の秘密鍵
`0b6366519a40eb4f384f7f84cf8bb716683ad1af8adbe60e59fe24ba042e396a` を使用する資格情報は、チュートリアル・
アプリケーションの [http://localhost:3000/credentials](http://localhost:3000/credentials) から生成できます。

3文字の claim である `iss`、`nbf`、`exp`、`sub` は [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519) に由来し、
claim の有効期限を制限するために `nbf` (not before) と `exp` (expiry date) を含めることができます。

![](https://fiware.github.io/tutorials.Verifiable-Credentials/img/create-claim.png)

#### 1️⃣ リクエスト:

```console
curl -L 'localhost:3000/vc/generate' \
-H 'Content-Type: application/json' \
-H 'Cookie: connect.sid=s%3AOb1s0q9UDOLwtLPs_xLMxP0aYTRD9wZQ.z5sNCOJ0IStsgf1f4C5AoDhtWZXVmjz7bmZiz2Ywi7k' \
--data-raw '{
    "key": "0b6366519a40eb4f384f7f84cf8bb716683ad1af8adbe60e59fe24ba042e396a",
    "iss": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
    "sub": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
    "nbf": 1754060243,
    "vc": {
        "@context": [
            "https://www.w3.org/2018/credentials/v1",
            "https://fiware.github.io/tutorials.Step-by-Step/credentials.jsonld"
        ],
        "type": [
            "VerifiableCredential",
            "OperatorCredential"
        ],
        "credentialSubject": {
            "firstName": "Alice",
            "lastName": "User",
            "eMail": "alice@test.com",
            "roles": [
                "OPERATOR"
            ]
        }
    }
}'
```

#### レスポンス:

レスポンスは **Alice** に渡される JWT トークンです - これは **Employee Badge** (社員証) を受け取ることに相当します。

```json
{
    "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vZml3YXJlLmdpdGh1Yi5pby90dXRvcmlhbHMuU3RlcC1ieS1TdGVwL2NyZWRlbnRpYWxzLmpzb25sZCJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiT3BlcmF0b3JDcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImZpcnN0TmFtZSI6IkFsaWNlIiwibGFzdE5hbWUiOiJVc2VyIiwiZU1haWwiOiJhbGljZUB0ZXN0LmNvbSIsInJvbGVzIjpbIk9QRVJBVE9SIl19fSwic3ViIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6YWxpY2UiLCJuYmYiOjE3NTQwNjAyNDMsImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFuaW1hbC13ZWxmYXJlIn0.YEoJtrpuR-bxDk-Y8yV0FPcCjtHkczq17vt4_iUV2D-kSbkAFjqkcAj5Vph48O4OeESI8GxoRZRH_yP-oat5hg"
}
```

## Verifiable Presentation の生成

**Vet** のところへ行くと、**Alice** は本当に **Animal Welfare** で働いているかどうかを確認されるため、Verifiable
Presentation の中に1つ以上の資格情報を提示する必要があります。各資格情報は JWT トークンの形式を取ります。この場合、
発行者 `iss` は **Alice** 自身であり、彼女はサブジェクト `sub` でもあります。通常、これらの Presentation には、
中間者攻撃の可能性を防ぐために、近い将来の `exp` が設定されています。

![](https://fiware.github.io/tutorials.Verifiable-Credentials/img/create-presentation.png)

#### 2️⃣ リクエスト:

```console
curl -L 'localhost:3000/vp/generate' \
-H 'Content-Type: application/json' \
-H 'Cookie: connect.sid=s%3AOb1s0q9UDOLwtLPs_xLMxP0aYTRD9wZQ.z5sNCOJ0IStsgf1f4C5AoDhtWZXVmjz7bmZiz2Ywi7k' \
--data-raw '{
    "iss": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
    "sub": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
    "payload": {
        "@context": [
            "https://www.w3.org/2018/credentials/v1"
        ],
        "type": [
            "VerifiablePresentation"
        ],
        "verifiableCredential": [
            "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vZml3YXJlLmdpdGh1Yi5pby90dXRvcmlhbHMuU3RlcC1ieS1TdGVwL2NyZWRlbnRpYWxzLmpzb25sZCJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiT3BlcmF0b3JDcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImZpcnN0TmFtZSI6IkFsaWNlIiwibGFzdE5hbWUiOiJVc2VyIiwiZU1haWwiOiJhbGljZUB0ZXN0LmNvbSIsInJvbGVzIjpbIk9QRVJBVE9SIl19fSwic3ViIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6YWxpY2UiLCJuYmYiOjE3NTQwNjAyNDMsImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFuaW1hbC13ZWxmYXJlIn0.YEoJtrpuR-bxDk-Y8yV0FPcCjtHkczq17vt4_iUV2D-kSbkAFjqkcAj5Vph48O4OeESI8GxoRZRH_yP-oat5hg"
        ]
    }
}'
```

#### レスポンス:

レスポンスは、また別の JWT トークンです。

```json
{
    "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2cCI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVQcmVzZW50YXRpb24iXSwidmVyaWZpYWJsZUNyZWRlbnRpYWwiOlsiZXlKaGJHY2lPaUpGVXpJMU5rc2lMQ0owZVhBaU9pSktWMVFpZlEuZXlKMll5STZleUpBWTI5dWRHVjRkQ0k2V3lKb2RIUndjem92TDNkM2R5NTNNeTV2Y21jdk1qQXhPQzlqY21Wa1pXNTBhV0ZzY3k5Mk1TSXNJbWgwZEhCek9pOHZabWwzWVhKbExtZHBkR2gxWWk1cGJ5OTBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3TDJOeVpXUmxiblJwWVd4ekxtcHpiMjVzWkNKZExDSjBlWEJsSWpwYklsWmxjbWxtYVdGaWJHVkRjbVZrWlc1MGFXRnNJaXdpVDNCbGNtRjBiM0pEY21Wa1pXNTBhV0ZzSWwwc0ltTnlaV1JsYm5ScFlXeFRkV0pxWldOMElqcDdJbVpwY25OMFRtRnRaU0k2SWtGc2FXTmxJaXdpYkdGemRFNWhiV1VpT2lKVmMyVnlJaXdpWlUxaGFXd2lPaUpoYkdsalpVQjBaWE4wTG1OdmJTSXNJbkp2YkdWeklqcGJJazlRUlZKQlZFOVNJbDE5ZlN3aWMzVmlJam9pWkdsa09uZGxZanBtYVhkaGNtVXVaMmwwYUhWaUxtbHZPblIxZEc5eWFXRnNjeTVUZEdWd0xXSjVMVk4wWlhBNllXeHBZMlVpTENKdVltWWlPakUzTlRRd05qQXlORE1zSW1semN5STZJbVJwWkRwM1pXSTZabWwzWVhKbExtZHBkR2gxWWk1cGJ6cDBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3T21GdWFXMWhiQzEzWld4bVlYSmxJbjAuWUVvSnRycHVSLWJ4RGstWTh5VjBGUGNDanRIa2N6cTE3dnQ0X2lVVjJELWtTYmtBRmpxa2NBajVWcGg0OE80T2VFU0k4R3hvUlpSSF95UC1vYXQ1aGciXX0sImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFsaWNlIn0.6_wuCNurZV5zawDKsPfJEEqWcmTpoTMG7r58HxAKJUkQB2bkRza2C7UoWOFu7DgHqDx9moSrQqrQ0n1Yp9JDDA"
}
```

## Verifiable Presentation の検証

Verifiable Presentation を受け取ると、**Vet** はまず、これが本当に **Alice** についての **Alice** 自身からの
Presentation であることを確認し、それが1つ以上の claim を保持する Verifiable Presentation であることを確認する
必要があります。

#### 3️⃣ リクエスト:

```console
curl -L 'localhost:3000/vp/verify' \
-H 'Content-Type: application/json' \
-d '{
    "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2cCI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVQcmVzZW50YXRpb24iXSwidmVyaWZpYWJsZUNyZWRlbnRpYWwiOlsiZXlKaGJHY2lPaUpGVXpJMU5rc2lMQ0owZVhBaU9pSktWMVFpZlEuZXlKMll5STZleUpBWTI5dWRHVjRkQ0k2V3lKb2RIUndjem92TDNkM2R5NTNNeTV2Y21jdk1qQXhPQzlqY21Wa1pXNTBhV0ZzY3k5Mk1TSXNJbWgwZEhCek9pOHZabWwzWVhKbExtZHBkR2gxWWk1cGJ5OTBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3TDJOeVpXUmxiblJwWVd4ekxtcHpiMjVzWkNKZExDSjBlWEJsSWpwYklsWmxjbWxtYVdGaWJHVkRjbVZrWlc1MGFXRnNJaXdpVDNCbGNtRjBiM0pEY21Wa1pXNTBhV0ZzSWwwc0ltTnlaV1JsYm5ScFlXeFRkV0pxWldOMElqcDdJbVpwY25OMFRtRnRaU0k2SWtGc2FXTmxJaXdpYkdGemRFNWhiV1VpT2lKVmMyVnlJaXdpWlUxaGFXd2lPaUpoYkdsalpVQjBaWE4wTG1OdmJTSXNJbkp2YkdWeklqcGJJazlRUlZKQlZFOVNJbDE5ZlN3aWMzVmlJam9pWkdsa09uZGxZanBtYVhkaGNtVXVaMmwwYUhWaUxtbHZPblIxZEc5eWFXRnNjeTVUZEdWd0xXSjVMVk4wWlhBNllXeHBZMlVpTENKdVltWWlPakUzTlRRd05qQXlORE1zSW1semN5STZJbVJwWkRwM1pXSTZabWwzWVhKbExtZHBkR2gxWWk1cGJ6cDBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3T21GdWFXMWhiQzEzWld4bVlYSmxJbjAuWUVvSnRycHVSLWJ4RGstWTh5VjBGUGNDanRIa2N6cTE3dnQ0X2lVVjJELWtTYmtBRmpxa2NBajVWcGg0OE80T2VFU0k4R3hvUlpSSF95UC1vYXQ1aGciXX0sImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFsaWNlIn0.6_wuCNurZV5zawDKsPfJEEqWcmTpoTMG7r58HxAKJUkQB2bkRza2C7UoWOFu7DgHqDx9moSrQqrQ0n1Yp9JDDA"
}'
```

#### レスポンス:

verifier は、`...JDDA` で終わる署名済み JWT が
[`https://fiware.github.io/tutorials.Step-by-Step/alice/did.json`](https://fiware.github.io/tutorials.Step-by-Step/alice/did.json)
にある公開鍵と一致することを確認します。つまり、Presentation が本当に **Alice** から来たものであることを確認します
- Verifiable Presentation は1つの claim (`-oat5hg` で終わる別の JWT) を保持しています。

```json
{
    "verified": true,
    "payload": {
        "vp": {
            "@context": ["https://www.w3.org/2018/credentials/v1"],
            "type": ["VerifiablePresentation"],
            "verifiableCredential": [
                "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vZml3YXJlLmdpdGh1Yi5pby90dXRvcmlhbHMuU3RlcC1ieS1TdGVwL2NyZWRlbnRpYWxzLmpzb25sZCJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiT3BlcmF0b3JDcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImZpcnN0TmFtZSI6IkFsaWNlIiwibGFzdE5hbWUiOiJVc2VyIiwiZU1haWwiOiJhbGljZUB0ZXN0LmNvbSIsInJvbGVzIjpbIk9QRVJBVE9SIl19fSwic3ViIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6YWxpY2UiLCJuYmYiOjE3NTQwNjAyNDMsImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFuaW1hbC13ZWxmYXJlIn0.YEoJtrpuR-bxDk-Y8yV0FPcCjtHkczq17vt4_iUV2D-kSbkAFjqkcAj5Vph48O4OeESI8GxoRZRH_yP-oat5hg"
            ]
        },
        "iss": "did:web:fiware.github.io:tutorials.Step-by-Step:alice"
    },
    "didResolutionResult": {
        "didDocument": {
            "@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/suites/jws-2020/v1"],
            "id": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
            "verificationMethod": [
                {
                    "id": "did:fiware.github.io:tutorials.Step-by-Step:alice#owner",
                    "type": "JsonWebKey2020",
                    "controller": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
                    "publicKeyJwk": {
                        "kty": "EC",
                        "crv": "secp256k1",
                        "x": "Nd3DeQ7G/1pTeYM6viWK6plbSD9E7cA9C2ONG9qG3CQ=",
                        "y": "LuMt0dFWni1/fs/VqfjNOHAZT3PWGxKU8kUlLffGtjM="
                    }
                }
            ],
            "authentication": ["did:web:fiware.github.io:tutorials.Step-by-Step:alice#owner"],
            "assertionMethod": ["did:web:fiware.github.io:tutorials.Step-by-Step:alice#owner"]
        },
        "didDocumentMetadata": {},
        "didResolutionMetadata": {
            "contentType": "application/did+ld+json"
        }
    },
    "issuer": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
    "signer": {
        "id": "did:fiware.github.io:tutorials.Step-by-Step:alice#owner",
        "type": "JsonWebKey2020",
        "controller": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
        "publicKeyJwk": {
            "kty": "EC",
            "crv": "secp256k1",
            "x": "Nd3DeQ7G/1pTeYM6viWK6plbSD9E7cA9C2ONG9qG3CQ=",
            "y": "LuMt0dFWni1/fs/VqfjNOHAZT3PWGxKU8kUlLffGtjM="
        }
    },
    "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2cCI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVQcmVzZW50YXRpb24iXSwidmVyaWZpYWJsZUNyZWRlbnRpYWwiOlsiZXlKaGJHY2lPaUpGVXpJMU5rc2lMQ0owZVhBaU9pSktWMVFpZlEuZXlKMll5STZleUpBWTI5dWRHVjRkQ0k2V3lKb2RIUndjem92TDNkM2R5NTNNeTV2Y21jdk1qQXhPQzlqY21Wa1pXNTBhV0ZzY3k5Mk1TSXNJbWgwZEhCek9pOHZabWwzWVhKbExtZHBkR2gxWWk1cGJ5OTBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3TDJOeVpXUmxiblJwWVd4ekxtcHpiMjVzWkNKZExDSjBlWEJsSWpwYklsWmxjbWxtYVdGaWJHVkRjbVZrWlc1MGFXRnNJaXdpVDNCbGNtRjBiM0pEY21Wa1pXNTBhV0ZzSWwwc0ltTnlaV1JsYm5ScFlXeFRkV0pxWldOMElqcDdJbVpwY25OMFRtRnRaU0k2SWtGc2FXTmxJaXdpYkdGemRFNWhiV1VpT2lKVmMyVnlJaXdpWlUxaGFXd2lPaUpoYkdsalpVQjBaWE4wTG1OdmJTSXNJbkp2YkdWeklqcGJJazlRUlZKQlZFOVNJbDE5ZlN3aWMzVmlJam9pWkdsa09uZGxZanBtYVhkaGNtVXVaMmwwYUhWaUxtbHZPblIxZEc5eWFXRnNjeTVUZEdWd0xXSjVMVk4wWlhBNllXeHBZMlVpTENKdVltWWlPakUzTlRRd05qQXlORE1zSW1semN5STZJbVJwWkRwM1pXSTZabWwzWVhKbExtZHBkR2gxWWk1cGJ6cDBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3T21GdWFXMWhiQzEzWld4bVlYSmxJbjAuWUVvSnRycHVSLWJ4RGstWTh5VjBGUGNDanRIa2N6cTE3dnQ0X2lVVjJELWtTYmtBRmpxa2NBajVWcGg0OE80T2VFU0k4R3hvUlpSSF95UC1vYXQ1aGciXX0sImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFsaWNlIn0.6_wuCNurZV5zawDKsPfJEEqWcmTpoTMG7r58HxAKJUkQB2bkRza2C7UoWOFu7DgHqDx9moSrQqrQ0n1Yp9JDDA",
    "policies": {},
    "verifiablePresentation": {
        "verifiableCredential": [
            {
                "credentialSubject": {
                    "firstName": "Alice",
                    "lastName": "User",
                    "eMail": "alice@test.com",
                    "roles": ["OPERATOR"],
                    "id": "did:web:fiware.github.io:tutorials.Step-by-Step:alice"
                },
                "issuer": {
                    "id": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare"
                },
                "type": ["VerifiableCredential", "OperatorCredential"],
                "@context": [
                    "https://www.w3.org/2018/credentials/v1",
                    "https://fiware.github.io/tutorials.Step-by-Step/credentials.jsonld"
                ],
                "issuanceDate": "2025-08-01T14:57:23.000Z",
                "proof": {
                    "type": "JwtProof2020",
                    "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vZml3YXJlLmdpdGh1Yi5pby90dXRvcmlhbHMuU3RlcC1ieS1TdGVwL2NyZWRlbnRpYWxzLmpzb25sZCJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiT3BlcmF0b3JDcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImZpcnN0TmFtZSI6IkFsaWNlIiwibGFzdE5hbWUiOiJVc2VyIiwiZU1haWwiOiJhbGljZUB0ZXN0LmNvbSIsInJvbGVzIjpbIk9QRVJBVE9SIl19fSwic3ViIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6YWxpY2UiLCJuYmYiOjE3NTQwNjAyNDMsImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFuaW1hbC13ZWxmYXJlIn0.YEoJtrpuR-bxDk-Y8yV0FPcCjtHkczq17vt4_iUV2D-kSbkAFjqkcAj5Vph48O4OeESI8GxoRZRH_yP-oat5hg"
                }
            }
        ],
        "holder": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
        "type": ["VerifiablePresentation"],
        "@context": ["https://www.w3.org/2018/credentials/v1"],
        "proof": {
            "type": "JwtProof2020",
            "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2cCI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVQcmVzZW50YXRpb24iXSwidmVyaWZpYWJsZUNyZWRlbnRpYWwiOlsiZXlKaGJHY2lPaUpGVXpJMU5rc2lMQ0owZVhBaU9pSktWMVFpZlEuZXlKMll5STZleUpBWTI5dWRHVjRkQ0k2V3lKb2RIUndjem92TDNkM2R5NTNNeTV2Y21jdk1qQXhPQzlqY21Wa1pXNTBhV0ZzY3k5Mk1TSXNJbWgwZEhCek9pOHZabWwzWVhKbExtZHBkR2gxWWk1cGJ5OTBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3TDJOeVpXUmxiblJwWVd4ekxtcHpiMjVzWkNKZExDSjBlWEJsSWpwYklsWmxjbWxtYVdGaWJHVkRjbVZrWlc1MGFXRnNJaXdpVDNCbGNtRjBiM0pEY21Wa1pXNTBhV0ZzSWwwc0ltTnlaV1JsYm5ScFlXeFRkV0pxWldOMElqcDdJbVpwY25OMFRtRnRaU0k2SWtGc2FXTmxJaXdpYkdGemRFNWhiV1VpT2lKVmMyVnlJaXdpWlUxaGFXd2lPaUpoYkdsalpVQjBaWE4wTG1OdmJTSXNJbkp2YkdWeklqcGJJazlRUlZKQlZFOVNJbDE5ZlN3aWMzVmlJam9pWkdsa09uZGxZanBtYVhkaGNtVXVaMmwwYUhWaUxtbHZPblIxZEc5eWFXRnNjeTVUZEdWd0xXSjVMVk4wWlhBNllXeHBZMlVpTENKdVltWWlPakUzTlRRd05qQXlORE1zSW1semN5STZJbVJwWkRwM1pXSTZabWwzWVhKbExtZHBkR2gxWWk1cGJ6cDBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3T21GdWFXMWhiQzEzWld4bVlYSmxJbjAuWUVvSnRycHVSLWJ4RGstWTh5VjBGUGNDanRIa2N6cTE3dnQ0X2lVVjJELWtTYmtBRmpxa2NBajVWcGg0OE80T2VFU0k4R3hvUlpSSF95UC1vYXQ1aGciXX0sImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFsaWNlIn0.6_wuCNurZV5zawDKsPfJEEqWcmTpoTMG7r58HxAKJUkQB2bkRza2C7UoWOFu7DgHqDx9moSrQqrQ0n1Yp9JDDA"
        }
    }
}
```

## Verifiable Credential の検証

`-oat5hg` で終わる JWT は、同様にデコードして検証できる Verifiable Credential です。この場合、**Animal Welfare**
(`did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare`) によって発行・署名された資格情報であり、
サブジェクトは `did:web:fiware.github.io:tutorials.Step-by-Step:alice` であることがわかります。

#### 4️⃣ リクエスト:

```console
curl -L 'localhost:3000/vc/verify' \
-H 'Content-Type: application/json' \
-H 'Cookie: connect.sid=s%3AskU1U3VI7mOAriJ7wd1-nV7DrfPNhOir.dVTi9sdMEtEv2Jlh5kACZvffzr%2FpDi5qmeGUotw38bc' \
-d '{
    "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vZml3YXJlLmdpdGh1Yi5pby90dXRvcmlhbHMuU3RlcC1ieS1TdGVwL2NyZWRlbnRpYWxzLmpzb25sZCJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiT3BlcmF0b3JDcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImZpcnN0TmFtZSI6IkFsaWNlIiwibGFzdE5hbWUiOiJVc2VyIiwiZU1haWwiOiJhbGljZUB0ZXN0LmNvbSIsInJvbGVzIjpbIk9QRVJBVE9SIl19fSwic3ViIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6YWxpY2UiLCJuYmYiOjE3NTQwNjAyNDMsImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFuaW1hbC13ZWxmYXJlIn0.YEoJtrpuR-bxDk-Y8yV0FPcCjtHkczq17vt4_iUV2D-kSbkAFjqkcAj5Vph48O4OeESI8GxoRZRH_yP-oat5hg"
}'
```

#### レスポンス:

```json
{
    "verified": true,
    "payload": {
        "vc": {
            "@context": [
                "https://www.w3.org/2018/credentials/v1",
                "https://fiware.github.io/tutorials.Step-by-Step/credentials.jsonld"
            ],
            "type": ["VerifiableCredential", "OperatorCredential"],
            "credentialSubject": {
                "firstName": "Alice",
                "lastName": "User",
                "eMail": "alice@test.com",
                "roles": ["OPERATOR"]
            }
        },
        "sub": "did:web:fiware.github.io:tutorials.Step-by-Step:alice",
        "nbf": 1754060243,
        "iss": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare"
    },
    "didResolutionResult": {
        "didDocument": {
            "@context": ["https://www.w3.org/ns/did/v1", "https://w3id.org/security/suites/jws-2020/v1"],
            "id": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
            "verificationMethod": [
                {
                    "id": "did:fiware.github.io:tutorials.Step-by-Step:animal-welfare#owner",
                    "type": "JsonWebKey2020",
                    "controller": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
                    "publicKeyJwk": {
                        "kty": "EC",
                        "crv": "secp256k1",
                        "x": "Nd3DeQ7G/1pTeYM6viWK6plbSD9E7cA9C2ONG9qG3CQ=",
                        "y": "LuMt0dFWni1/fs/VqfjNOHAZT3PWGxKU8kUlLffGtjM="
                    }
                }
            ],
            "authentication": ["did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare#owner"],
            "assertionMethod": ["did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare#owner"]
        },
        "didDocumentMetadata": {},
        "didResolutionMetadata": {
            "contentType": "application/did+ld+json"
        }
    },
    "issuer": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
    "signer": {
        "id": "did:fiware.github.io:tutorials.Step-by-Step:animal-welfare#owner",
        "type": "JsonWebKey2020",
        "controller": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
        "publicKeyJwk": {
            "kty": "EC",
            "crv": "secp256k1",
            "x": "Nd3DeQ7G/1pTeYM6viWK6plbSD9E7cA9C2ONG9qG3CQ=",
            "y": "LuMt0dFWni1/fs/VqfjNOHAZT3PWGxKU8kUlLffGtjM="
        }
    },
    "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vZml3YXJlLmdpdGh1Yi5pby90dXRvcmlhbHMuU3RlcC1ieS1TdGVwL2NyZWRlbnRpYWxzLmpzb25sZCJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiT3BlcmF0b3JDcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImZpcnN0TmFtZSI6IkFsaWNlIiwibGFzdE5hbWUiOiJVc2VyIiwiZU1haWwiOiJhbGljZUB0ZXN0LmNvbSIsInJvbGVzIjpbIk9QRVJBVE9SIl19fSwic3ViIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6YWxpY2UiLCJuYmYiOjE3NTQwNjAyNDMsImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFuaW1hbC13ZWxmYXJlIn0.YEoJtrpuR-bxDk-Y8yV0FPcCjtHkczq17vt4_iUV2D-kSbkAFjqkcAj5Vph48O4OeESI8GxoRZRH_yP-oat5hg",
    "policies": {},
    "verifiableCredential": {
        "credentialSubject": {
            "firstName": "Alice",
            "lastName": "User",
            "eMail": "alice@test.com",
            "roles": ["OPERATOR"],
            "id": "did:web:fiware.github.io:tutorials.Step-by-Step:alice"
        },
        "issuer": {
            "id": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare"
        },
        "type": ["VerifiableCredential", "OperatorCredential"],
        "@context": [
            "https://www.w3.org/2018/credentials/v1",
            "https://fiware.github.io/tutorials.Step-by-Step/credentials.jsonld"
        ],
        "issuanceDate": "2025-08-01T14:57:23.000Z",
        "proof": {
            "type": "JwtProof2020",
            "jwt": "eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vZml3YXJlLmdpdGh1Yi5pby90dXRvcmlhbHMuU3RlcC1ieS1TdGVwL2NyZWRlbnRpYWxzLmpzb25sZCJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiT3BlcmF0b3JDcmVkZW50aWFsIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImZpcnN0TmFtZSI6IkFsaWNlIiwibGFzdE5hbWUiOiJVc2VyIiwiZU1haWwiOiJhbGljZUB0ZXN0LmNvbSIsInJvbGVzIjpbIk9QRVJBVE9SIl19fSwic3ViIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6YWxpY2UiLCJuYmYiOjE3NTQwNjAyNDMsImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFuaW1hbC13ZWxmYXJlIn0.YEoJtrpuR-bxDk-Y8yV0FPcCjtHkczq17vt4_iUV2D-kSbkAFjqkcAj5Vph48O4OeESI8GxoRZRH_yP-oat5hg"
        }
    }
}
```

### Data Space 内での Verifiable Credential の使用

**Alice** が Verifiable Credential を受け取ったので、彼女はそれを使って Data Space 内で Operator のロールを主張し、
獣医の記録 (Veterinary Records) へのアクセスを得ることができます。トークンを持たずに記録にアクセスしようとする最初の
試みはエラーになり、verifier がポート `1030` に存在することが示されます。

#### Verifiable Credential なしで獣医の記録にアクセスする

#### 5️⃣ リクエスト:

```console
curl -L 'localhost:1030/ngsi-ld/v1/entities?local=true' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"
```

#### レスポンス:

レスポンスは、以下の内容を伴う **401 - Unauthorized** エラー・コードです。

```json
{
    "type": "urn:dx:as:MissingAuthenticationToken",
    "title": "Unauthorized",
    "detail": "message"
}
```

### 無効な Verifiable Credential で獣医の記録にアクセスする

Verifiable Credential は Bearer トークンとして `Authorization` ヘッダに追加されます。Bearer トークンは JWT であり、
デコードされて検証されます - Bearer トークンの内容が主張された発行者と一致しない場合、トークンは拒否されます。

#### 6️⃣ リクエスト:

```console
curl -L 'localhost:1030/ngsi-ld/v1/entities?local=true' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Authorization: Bearer eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2cCI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVQcmVzZW50YXRpb24iXSwidmVyaWZpYWJsZUNyZWRlbnRpYWwiOlsiZXlKaGJHY2lPaUpGVXpJMU5rc2lMQ0owZVhBaU9pSktWMVFpZlEuZXlKMll5STZleUpBWTI5dWRHVjRkQ0k2V3lKb2RIUndjem92TDNkM2R5NTNNeTV2Y21jdk1qQXhPQzlqY21Wa1pXNTBhV0ZzY3k5Mk1TSXNJbWgwZEhCek9pOHZabWwzWVhKbExtZHBkR2gxWWk1cGJ5OTBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3TDJOeVpXUmxiblJwWVd4ekxtcHpiMjVzWkNKZExDSjBlWEJsSWpwYklsWmxjbWxtYVdGaWJHVkRjbVZrWlc1MGFXRnNJaXdpUkhKcGRtVnljMHhwWTJWdWMyVWlYU3dpWTNKbFpHVnVkR2xoYkZOMVltcGxZM1FpT25zaWFXUWlPaUoxY200NlpISnBkbVZ5Y3kxc2FXTmxibk5sT21Gc2FXTmxPakF3TVNJc0ltNWhiV1VpT2lKQmJHbGpaU0lzSW1SaGRHVlBaa0pwY25Sb0lqb2lNVGs0TkMwd09TMHhOeUlzSW5Cc1lXTmxUMlpDYVhKMGFDSTZJa0psY214cGJpSXNJbVJoZEdWUFprbHpjM1ZsSWpvaU1qQXdOeTB3TVMwd09TSXNJbVJoZEdWUFprVjRjR2x5ZVNJNklqSXdNemN0TURFdE1Ea2lMQ0pwYzNOMWFXNW5RWFYwYUc5eWFYUjVJam9pUkZaTVFTSXNJbXhwWTJWdWMyVk9kVzFpWlhJaU9pSkJURWxEUlRFeU16UTFXRmc1U1Vvek5TSXNJblpsYUdsamJHVkRZWFJsWjI5eWFXVnpJanBiSWtJaUxDSkNNU0lzSWtNaVhYMTlMQ0p6ZFdJaU9pSmthV1E2ZDJWaU9tWnBkMkZ5WlM1bmFYUm9kV0l1YVc4NmRIVjBiM0pwWVd4ekxsTjBaWEF0WW5rdFUzUmxjRHBoYkdsalpTSXNJbTVpWmlJNmJuVnNiQ3dpYVhOeklqb2laR2xrT25kbFlqcG1hWGRoY21VdVoybDBhSFZpTG1sdk9uUjFkRzl5YVdGc2N5NVRkR1Z3TFdKNUxWTjBaWEE2WjI5MkluMC5peUxJaG5Bd3ZzbU90QnVXd3Jid0FSRXVPY0plblZYeUNVQ1dlNk1qakl6NDJqNi1XcVhseE05bk1xV25QeXQwVG92MGFSeTBqSG5KVUFPRVU0TjlaUSJdfSwiaXNzIjoiZGlkOndlYjpmaXdhcmUuZ2l0aHViLmlvOnR1dG9yaWFscy5TdGVwLWJ5LVN0ZXA6Z292In0.PTHHUoGjAT9n_DQukoxYCVZ0o9yjZJGiTBWQ3kI9QxdO1D-TkbBdBRfhzo4-ezRnW4BFpKkse1fsdb_FymtgCw' \
-H 'Cookie: connect.sid=s%3AfQyNTuX_bUcm7dPusUIRHehr0myIcchy.DvjkMq2W94uKRAIAtCjrz5ZCB52ulI8jB2rMbiWnvwc'


```

#### レスポンス:

```json
{
    "type": "urn:dx:as:InvalidAuthenticationToken",
    "title": "Unauthorized",
    "detail": "invalid_signature: no matching public key found"
}
```

資格情報が拒否された場合、レスポンスは、以下の内容を伴う **401 - Unauthorized** エラー・コードになります。

実際の Credential Verifier は、資格情報を主張しているすべての発行者が本当に各 Verifiable Credential に署名している
ことを確認するだけでなく、`exp` と `nbf` が範囲内であることも確認する点に注意してください。

### 有効な Verifiable Credential で獣医の記録にアクセスする

適切な Verifiable Presentation があれば、**Animal** の記録にアクセスできます :

#### 7️⃣ リクエスト:

```console
curl -L 'localhost:1030/ngsi-ld/v1/entities?local=true' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Authorization: Bearer eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2cCI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVQcmVzZW50YXRpb24iXSwidmVyaWZpYWJsZUNyZWRlbnRpYWwiOlsiZXlKaGJHY2lPaUpGVXpJMU5rc2lMQ0owZVhBaU9pSktWMVFpZlEuZXlKMll5STZleUpBWTI5dWRHVjRkQ0k2V3lKb2RIUndjem92TDNkM2R5NTNNeTV2Y21jdk1qQXhPQzlqY21Wa1pXNTBhV0ZzY3k5Mk1TSXNJbWgwZEhCek9pOHZabWwzWVhKbExtZHBkR2gxWWk1cGJ5OTBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3TDJOeVpXUmxiblJwWVd4ekxtcHpiMjVzWkNKZExDSjBlWEJsSWpwYklsWmxjbWxtYVdGaWJHVkRjbVZrWlc1MGFXRnNJaXdpVDNCbGNtRjBiM0pEY21Wa1pXNTBhV0ZzSWwwc0ltTnlaV1JsYm5ScFlXeFRkV0pxWldOMElqcDdJbVpwY25OMFRtRnRaU0k2SWtGc2FXTmxJaXdpYkdGemRFNWhiV1VpT2lKVmMyVnlJaXdpWlUxaGFXd2lPaUpoYkdsalpVQjBaWE4wTG1OdmJTSXNJbkp2YkdWeklqcGJJazlRUlZKQlZFOVNJbDE5ZlN3aWMzVmlJam9pWkdsa09uZGxZanBtYVhkaGNtVXVaMmwwYUhWaUxtbHZPblIxZEc5eWFXRnNjeTVUZEdWd0xXSjVMVk4wWlhBNllXeHBZMlVpTENKdVltWWlPakUzTlRRd05qQXlORE1zSW1semN5STZJbVJwWkRwM1pXSTZabWwzWVhKbExtZHBkR2gxWWk1cGJ6cDBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3T21GdWFXMWhiQzEzWld4bVlYSmxJbjAuWUVvSnRycHVSLWJ4RGstWTh5VjBGUGNDanRIa2N6cTE3dnQ0X2lVVjJELWtTYmtBRmpxa2NBajVWcGg0OE80T2VFU0k4R3hvUlpSSF95UC1vYXQ1aGciXX0sImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFsaWNlIn0.6_wuCNurZV5zawDKsPfJEEqWcmTpoTMG7r58HxAKJUkQB2bkRza2C7UoWOFu7DgHqDx9moSrQqrQ0n1Yp9JDDA'

```

#### レスポンス:

```json
[
    {
        "id": "urn:ngsi-ld:Animal:cow006",
        "type": "Animal",
        "fedWith": { "type": "Property", "value": "Oats"},
        "species": { "type": "Property", "value": "dairy cattle"},
        "name": { "type": "Property", "value": "Twilight"},
        "sex": { "type": "VocabProperty", "vocab": "Female"},
        "phenologicalCondition": { "type": "VocabProperty", "vocab": "femaleAdult"},
        "healthCondition": {
            "type": "VocabProperty",
            "vocab": "healthy",
            "observedAt": "2024-02-02T15:00:00.000Z"
        },
        "reproductiveCondition": {
           "type": "VocabProperty",
            "vocab": "noStatus",
            "observedAt": "2024-02-02T15:00:00.000Z"
        }
    },
    ... etc
]
```

レスポンスには一連の **Animal** レコードが含まれていますが、`http://localhost:3000/vp/monitor` にある
[Verifiable Presentation Monitor](http://localhost:3000/vp/monitor) で出力を確認すると、以下の出力が見つかります :

```
OperatorCredential issued by did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare was NOT TRUSTED
```

これは、実際にはさらなる確認が必要なためです。Verifiable Credential が発行者によって署名されているだけでなく、その
発行者が Data Space 内で資格情報の有効な発行者である必要があります。verifier がこれを確認する方法は、trusted issuers
list に問い合わせることです。このリストの場所は、Verifiable Credentials verifier に関連付けられた configuration
service 内で定義されています。

### Trusted Issuer の確認

configuration service はポート 8081 で実行されています。vet にとって有効な発行者の一覧は、service リクエストを
行うことで見つけることができます。

#### 8️⃣ リクエスト:

```console
curl -L 'localhost:8081/service/vet'
```

#### レスポンス:

```json
{
    "id": "vet",
    "defaultOidcScope": "default",
    "oidcScopes": {
        "default": {
            "credentials": [
                {
                    "type": "VerifiableCredential",
                    "trustedParticipantsLists": [],
                    "trustedIssuersLists": ["http://trusted-issuers-list:8080"],
                    "holderVerification": {
                        "enabled": false,
                        "claim": "subject"
                    },
                    "requireCompliance": false,
                    "jwtInclusion": {
                        "enabled": true,
                        "fullInclusion": false,
                        "claimsToInclude": []
                    }
                }
            ],
            "presentationDefinition": {
                "id": null,
                "input_descriptors": null
            },
            "flatClaims": false
        }
    }
}
```

レスポンスは、Verifiable Credential が `http://trusted-issuers-list:8080` にある trusted issuers list に照らして
確認できることを示しています。

### Trusted Issuers List の読み取り

trusted issuers list は、通常 Data Space の運営者によって管理されています。誰が有効なユーザであるか、そしてその
発行者がどのようなアクションを生成できるかについての情報を保持しています。trusted issuers list はポート 8080 で
実行されているものが見つかります - 最初は有効な発行者は存在しません。

#### 9️⃣ リクエスト:

```console
curl -L 'localhost:8080/v4/issuers'
```

#### レスポンス:

```json
{
    "self": "/v4/issuers/",
    "items": [],
    "total": 0,
    "pageSize": 0,
    "links": null
}
```

### Trusted Issuers List への Trusted Issuer の追加

trusted issuer を追加するには、`/issuer` エンドポイントに **POST** リクエストを行います。ここでは、発行者が
`did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare` であり、その組織が `OPERATOR` と `VISITOR` という
2つの異なるロールを持つ **OperatorCredentials** を作成できることがわかります。

#### 1️⃣0️⃣ リクエスト:

```console
curl -L 'localhost:8080/issuer' \
-H 'Content-Type: application/json' \
-d '{
  "did": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
  "credentials": [
    {
      "validFor": {
        "from": "2017-07-21T17:32:28Z",
        "to": "2023-07-21T17:32:28Z"
      },
      "credentialsType": "OperatorCredential",
      "claims": [
        {
          "name": "roles",
          "allowedValues": [
            "OPERATOR",
            "VISITOR"
          ]
        }
      ]
    }
  ]
}'
```

### Trusted Issuers List からの読み取り

この trusted issuers list は、2つの異なる形式で発行者の権利を取得できます。まず、
`/issuer/did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare` エンドポイントに **GET** リクエストを行う
ことで、プレーン・テキストの発行者情報を取得します。

#### 1️⃣1️⃣ リクエスト:

```console
curl -L 'localhost:8080/issuer/did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare'
```

#### レスポンス:

レスポンスは以下のとおりです。

```json
{
    "did": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
    "credentials": [
        {
            "credentialsType": "OperatorCredential",
            "claims": [
                {
                    "name": "roles",
                    "allowedValues": ["OPERATOR", "VISITOR"]
                }
            ]
        }
    ]
}
```

#### 1️⃣2️⃣ リクエスト:

trusted issuers list は、[EBSI 互換](https://hub.ebsi.eu/#/) 形式でも発行者データを取得できます。

```console
curl -L 'localhost:8080/v4/issuers/did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare'
```

#### レスポンス:

レスポンスは以下のとおりです。ここで `hash` と `body` は、それぞれペイロード本体の sha256 ハッシュと base64
エンコードされた文字列です。

```json
{
    "did": "did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare",
    "attributes": [
        {
            "hash": "LIayBgwZ84KzjTIe9bHQfKE1/NRJIhPHrWE3NUiwuBI=",
            "body": "eyJjcmVkZW50aWFsc1R5cGUiOiJPcGVyYXRvckNyZWRlbnRpYWwiLCJjbGFpbXMiOlt7Im5hbWUiOiJyb2xlcyIsImFsbG93ZWRWYWx1ZXMiOlsiT1BFUkFUT1IiLCJWSVNJVE9SIl19XX0=",
            "issuerType": "Undefined"
        }
    ]
}
```

さて、適切な Verifiable Presentation があれば、**Animal** の記録にアクセスできます :

#### 1️⃣2️⃣ リクエスト:

```console
curl -L 'localhost:1030/ngsi-ld/v1/entities?local=true' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Authorization: Bearer eyJhbGciOiJFUzI1NksiLCJ0eXAiOiJKV1QifQ.eyJ2cCI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVQcmVzZW50YXRpb24iXSwidmVyaWZpYWJsZUNyZWRlbnRpYWwiOlsiZXlKaGJHY2lPaUpGVXpJMU5rc2lMQ0owZVhBaU9pSktWMVFpZlEuZXlKMll5STZleUpBWTI5dWRHVjRkQ0k2V3lKb2RIUndjem92TDNkM2R5NTNNeTV2Y21jdk1qQXhPQzlqY21Wa1pXNTBhV0ZzY3k5Mk1TSXNJbWgwZEhCek9pOHZabWwzWVhKbExtZHBkR2gxWWk1cGJ5OTBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3TDJOeVpXUmxiblJwWVd4ekxtcHpiMjVzWkNKZExDSjBlWEJsSWpwYklsWmxjbWxtYVdGaWJHVkRjbVZrWlc1MGFXRnNJaXdpVDNCbGNtRjBiM0pEY21Wa1pXNTBhV0ZzSWwwc0ltTnlaV1JsYm5ScFlXeFRkV0pxWldOMElqcDdJbVpwY25OMFRtRnRaU0k2SWtGc2FXTmxJaXdpYkdGemRFNWhiV1VpT2lKVmMyVnlJaXdpWlUxaGFXd2lPaUpoYkdsalpVQjBaWE4wTG1OdmJTSXNJbkp2YkdWeklqcGJJazlRUlZKQlZFOVNJbDE5ZlN3aWMzVmlJam9pWkdsa09uZGxZanBtYVhkaGNtVXVaMmwwYUhWaUxtbHZPblIxZEc5eWFXRnNjeTVUZEdWd0xXSjVMVk4wWlhBNllXeHBZMlVpTENKdVltWWlPakUzTlRRd05qQXlORE1zSW1semN5STZJbVJwWkRwM1pXSTZabWwzWVhKbExtZHBkR2gxWWk1cGJ6cDBkWFJ2Y21saGJITXVVM1JsY0MxaWVTMVRkR1Z3T21GdWFXMWhiQzEzWld4bVlYSmxJbjAuWUVvSnRycHVSLWJ4RGstWTh5VjBGUGNDanRIa2N6cTE3dnQ0X2lVVjJELWtTYmtBRmpxa2NBajVWcGg0OE80T2VFU0k4R3hvUlpSSF95UC1vYXQ1aGciXX0sImlzcyI6ImRpZDp3ZWI6Zml3YXJlLmdpdGh1Yi5pbzp0dXRvcmlhbHMuU3RlcC1ieS1TdGVwOmFsaWNlIn0.6_wuCNurZV5zawDKsPfJEEqWcmTpoTMG7r58HxAKJUkQB2bkRza2C7UoWOFu7DgHqDx9moSrQqrQ0n1Yp9JDDA'

```

#### レスポンス:

```json
[
    {
        "id": "urn:ngsi-ld:Animal:cow006",
        "type": "Animal",
        "fedWith": { "type": "Property", "value": "Oats"},
        "species": { "type": "Property", "value": "dairy cattle"},
        "name": { "type": "Property", "value": "Twilight"},
        "sex": { "type": "VocabProperty", "vocab": "Female"},
        "phenologicalCondition": { "type": "VocabProperty", "vocab": "femaleAdult"},
        "healthCondition": {
            "type": "VocabProperty",
            "vocab": "healthy",
            "observedAt": "2024-02-02T15:00:00.000Z"
        },
        "reproductiveCondition": {
           "type": "VocabProperty",
            "vocab": "noStatus",
            "observedAt": "2024-02-02T15:00:00.000Z"
        }
    },
    ... etc
]
```

レスポンスには一連の **Animal** レコードが含まれており、`http://localhost:3000/vp/monitor` にある
[Verifiable Presentation Monitor](http://localhost:3000/vp/monitor) で出力を確認すると、以下の出力が見つかります :

```text
The following claims were made [{"name":"roles","allowedValues":["VISITOR","OPERATOR"]}]

{
  "firstName": "Alice",
  "lastName": "User",
  "eMail": "alice@test.com",
  "roles": [
    "OPERATOR"
  ],
  "id": "did:web:fiware.github.io:tutorials.Step-by-Step:alice"
}
```

ご覧のとおり、`role:OPERATOR` は `did:web:fiware.github.io:tutorials.Step-by-Step:animal-welfare` が作成できる
有効な設定です。これらの値を照合することで、実際の Data Space Connector は、自身の PEP を使ってアクセスを許可または
拒否できるようになります。

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://ngsi-ld-tutorials.rtfd.io)を読むことで見つけることができます

## License

[MIT](LICENSE) © 2025-2026 FIWARE Foundation e.V.
