---
layout: post
title: "[NestJS] MongoDB 연결하기"
categories: NestJS
tags: MongoDB
---

![1*k4mDCvHq5LlRgSRCs69c2Q](https://user-images.githubusercontent.com/108377235/202498521-23b991a2-fd16-4adf-b733-25c9b80f9296.jpeg)

<br/>

## Intro

이번에 진행하는 프로젝트에서도 **백엔드 파트**를 맡게 되었습니다.

지난번 프로젝트에서는 `Django`를 사용해 백엔드 서버를 구축하였지만

이번 프로젝트는 `TypeScript`를 학습하고 사용해보고 싶어서 팀원분께 말씀드렸고

흔쾌히 수락해주신 덕분에 `TypeScript`를 사용하게 되었습니다.

<br/>

`Express.js`에서도 TypeScript를 사용할 수는 있지만

설정이 번거롭고 조금 귀찮은 부분도 많다는 후기를 찾아볼 수 있었습니다.

그래서 TypeScript를 기본으로 사용하는 `NestJS`를 사용해서 백엔드를 구축해보기로 하였습니다.

`NestJS`에 대한 기본적인 내용은 추후 블로그에 작성할 수 있도록 하고,

우선 프로젝트를 진행하면서 기억해둘만한 것이나 문제를 해결했던 경험 위주로 작성을 하겠습니다.

<br/>

우선 [NestJS로 배우는 백엔드 프로그래밍](https://wikidocs.net/book/7059) 을 통해 NestJS의 기본적인 부분을 학습하고 있습니다.

여기에서 데이터베이스를 연결할 때 `TypeORM`을 사용해 `MySQL`을 예시로 설명하고 있었습니다.

이번 프로젝트에서 MySQL을 사용할 지, MongoDB를 사용할 지 정해지지 않은 시점이기 때문에

예시에 나와있는 것이 아닌 `MongoDB`를 연결하는 방법을 학습해보았습니다.

<br/>

## Start

우선 이 글의 내용은 다음과 같은 `전제조건`을 가지고 있습니다.

- [NestJS로 배우는 백엔드 프로그래밍](https://wikidocs.net/book/7059) 7장까지 진행
- MongoDB Atlas 설정 (local도 가능)

만약, 다음과 같은 조건이 아니더라도 아래 코드가 도움이 되실 수 있으니

준비가 다 되었다면 차근차근 따라오시면 됩니다.

<br/>

단, 단순히 코드를 copy/paste 하는 것이 아니라 코드를 이해하면서 봐주시면 감사하겠습니다.

<br/>

### Install Mongoose

다음 코드를 입력하여 `Mongoose`를 설치합니다.

```bash
$ npm install --save @nestjs/mongoose mongoose
```

<br/>

### Create Schema

다음 코드를 입력하여 `user Schema`를 생성합니다.

```bash
$ nest g s user
```

<br/>

### user.schema.ts

위 코드를 통해 생성된 `../schemas/user.schema.ts` 파일에 다음과 같은 코드를 입력합니다.

```tsx
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import mongoose, { Document } from "mongoose";

export type UserDocument = User & Document;

@Schema({ timestamps: { createdAt: "createdAt", updatedAt: "updatedAt" } })
export class User {
  @Prop({ default: new Date(), type: mongoose.Schema.Types.Date })
  createdAt: Date;

  @Prop({ default: new Date(), type: mongoose.Schema.Types.Date })
  updatedAt: Date;

  @Prop()
  id: string;

  @Prop({ required: true })
  name: string;

  @Prop({ required: true })
  email: string;

  @Prop({ required: true })
  password: string;

  @Prop()
  signupVerifyToken: string;
}

export const UserSchema = SchemaFactory.createForClass(User);
```

<br/>

### app.module.ts

`app.module.ts` 파일에 다음과 같은 코드를 추가합니다.

```tsx
...
import { MongooseModule } from '@nestjs/mongoose';
...

@Module({
	imports: [
		...
		MongooseModule.forRoot('mongodb address'),
		...
	],
	...
})
```

<br/>

### users.module.ts

`users.module.ts` 파일에 다음과 같은 코드를 추가합니다.

```tsx
...
import { MongooseModule } from '@nestjs/mongoose';
import { User, UserSchema } from './schemas/user.schema';
...

@Module({
	imports: [
		...
    MongooseModule.forFeature([{ name: User.name, schema: UserSchema }]),
		...
  ],
	...
})
```

<br/>

### users.service.ts

마지막으로 `users.service.ts` 파일에 다음과 같은 코드를 추가합니다.

이는 [NestJS로 배우는 백엔드 프로그래밍 8.3장](https://wikidocs.net/158617) 의 예시와 같은 동작을 하는 `MongoDB` 설정입니다.

```tsx
...
import { User, UserDocument } from './schemas/user.schema';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
...

@Injectable()
export class UsersService {
	constructor(
		...
    @InjectModel(User.name) private userModel: Model<UserDocument>,
  ) {}

	...

	private async checkUserExists(email: string): Promise<boolean> {
    const user = await this.userModel.findOne({ email: email });
    return user !== null;
  }

	...

	private async saveUser(
    name: string,
    email: string,
    password: string,
    signupVerifyToken: string,
  ) {
    const user = new this.userModel();

    user.id = ulid();
    user.name = name;
    user.email = email;
    user.password = password;
    user.signupVerifyToken = signupVerifyToken;

    await user.save();
  }
	...
}
```

---

<br/>

제가 작성한 코드와 완전히 다른 코드라도 `NestJS`에서 `Mongoose`로 `MongoDB`를 사용하는 핵심 코드는 동일하기 때문에

저와 같은 어려움을 겪는 개발자분들에게 조금이나마 도움이 될 수 있었으면 좋겠습니다.

또한, 위와 같이 Mongoose를 사용하는 방법이 아닌 `TypeORM` 을 사용해 MongoDB와 같은 데이터베이스를 연결하는 방법도 있기 때문에

추후에 기회가 된다면 `TypeORM`도 학습 후 다뤄보도록 하겠습니다.

<br/>

만약 코드에 개선점이나 수정할 점이 있다면 게시물 하단에 댓글로 작성해주시면 감사하겠습니다.
