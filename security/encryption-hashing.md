### Encryption and Hashing

**암호화(Encryption)**는 정보를 인코딩하는 과정입니다. 이 프로세스는 일반 텍스트로 알려진 정보의 원래 표현을 암호문으로 알려진 대체 형식으로 변환합니다. 이상적으로는 권한이 있는 당사자만이 암호문을 일반 텍스트로 해독하고 원래 정보에 액세스할 수 있습니다. 암호화는 자체적으로 간섭을 방지하지는 않지만 인터셉터가 될 수 있는 이해 가능한 콘텐츠를 거부합니다. 암호화는 양방향 기능입니다. 암호화된 것은 적절한 키로 해독할 수 있습니다.

**해싱(Hasing)**은 주어진 키를 다른 값으로 변환하는 프로세스입니다. 해시 함수는 수학적 알고리즘에 따라 새 값을 생성하는 데 사용됩니다. 해싱이 완료되면 출력에서 입력으로 이동할 수 없습니다.

#### Encryption

Node.js는 문자열, 숫자, 버퍼, 스트림등을 암호화하고 복호화하는 데 사용할 수있는 기본 제공 [crypto 모듈](https://nodejs.org/api/crypto.html)을 제공합니다. Nest 자체는 불필요한 추상화를 방지하기 위해 이 모듈 위에 추가 패키지를 제공하지 않습니다.

예를 들어 AES (Advanced Encryption System) `'aes-256-ctr'` 알고리즘 CTR 암호화 모드를 사용해 보겠습니다.

```typescript
import { createCipheriv, randomBytes } from 'crypto';
import { promisify } from 'util';

const iv = randomBytes(16);
const password = 'Password used to generate key';

// The key length is dependent on the algorithm.
// In this case for aes256, it is 32 bytes.
const key = (await promisify(scrypt)(password, 'salt', 32)) as Buffer;
const cipher = createCipheriv('aes-256-ctr', key, iv);

const textToEncrypt = 'Nest';
const encryptedText = Buffer.concat([
  cipher.update(textToEncrypt),
  cipher.final(),
]);
```

`encryptedText` 값을 해독하는 방법:

```typescript
import { createDecipheriv } from 'crypto';

const decipher = createDecipheriv('aes-256-ctr', key, iv);
const decryptedText = Buffer.concat([
  decipher.update(encryptedText),
  decipher.final(),
]);
```

#### Hashing

해싱의 경우 [bcrypt](https://www.npmjs.com/package/bcrypt) 또는 [argon2](https://www.npmjs.com/package/argon2) 패키지를 사용하는 것이 좋습니다. Nest 자체는 불필요한 추상화를 방지하기 위해 이러한 모듈 위에 추가 래퍼를 제공하지 않습니다 (학습 곡선이 짧아짐).

예를 들어 `bcrypt`를 사용하여 임의의 암호를 해시해 보겠습니다.

먼저 필수 패키지를 설치하십시오.

```shell
$ npm i bcrypt
$ npm i -D @types/bcrypt
```

설치가 완료되면 다음과 같이 `hash` 기능을 사용할 수 있습니다.

```typescript
import * as bcrypt from 'bcrypt';

const saltOrRounds = 10;
const password = 'random_password';
const hash = await bcrypt.hash(password, saltOrRounds);
```

솔트를 생성하려면 `genSalt` 함수를 사용하세요.

```typescript
const salt = await bcrypt.genSalt();
```

비밀번호를 비교/확인하려면 `compare` 기능을 사용하세요.

```typescript
const isMatch = await bcrypt.compare(password, hash);
```

사용 가능한 기능에 대한 자세한 내용은 [여기](https://www.npmjs.com/package/bcrypt)에서 확인할 수 있습니다.
