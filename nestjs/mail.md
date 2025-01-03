## Nestjs发送验证码实现指南

<aside>
本文详细介绍了在Nestjs中实现邮件验证码系统的完整解决方案，包括问题处理、配置说明和核心实现。
</aside>

# 1. 问题描述与解决方案

## 1.1 遇到的问题

> 使用@nestjs-modules/mailer发送验证码时，在打包后出现模板文件找不到的问题

问题原因：打包过程中template文件未被正确包含在构建输出中

![https://cdn.liboqiao.top/markdown/image-20241224150448485.png](https://cdn.liboqiao.top/markdown/image-20241224150448485.png))

但是本地有啊，齐了怪了

![https://cdn.liboqiao.top/markdown/image-20241224150603887.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/bb729ae4b34d4081abd0aefc6059994c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgcGlrYWNodeWGsuWGsuWGsg==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjk2MDMxMzQ5MTEyOTQ0NyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1735635682&x-orig-sign=HcJ92jz5m9XZAO1j2pfcEjJoPF4%3D)

好吧 我本地打个包看看

![https://cdn.liboqiao.top/markdown/image-20241224150710556.png](https://cdn.liboqiao.top/markdown/image-20241224150710556.png)

好吧 确实没有，猜想：是不是这个文件被引用 所以被干掉了，看下文档怎么办，了解到可以打包的时候 把该template文件夹 打包到 该email目录下

so 修改`nest-cli.json`中的`compilerOptions`

      "compilerOptions": {
        "assets": [
          {
            "include": "email/template/**/*.{ejs,html}",
            "outDir": "dist/src/"
          }
        ],
        "deleteOutDir": true
      }

确实这样就可以了

![https://cdn.liboqiao.top/markdown/image-20241224151229539.png](https://cdn.liboqiao.top/markdown/image-20241224151229539.png)

## 1.2 解决方案

通过修改nest-cli.json配置文件，确保模板文件被正确打包到目标目录：

```json
{
  "compilerOptions": {
    "assets": [
      {
        "include": "email/template/**/*.{ejs,html}",
        "outDir": "dist/src/"
      }
    ],
    "deleteOutDir": true
  }
}
```

# 2. 邮件发送配置详解

# 2. 邮件发送配置详解

## 2.1 MailerModule配置

<aside>
使用MailerModule进行邮件服务配置，支持以下关键特性：

</aside>

*   ✉️ 支持QQ邮箱服务
*   📝 使用EJS模板系统
*   🔒 支持安全传输配置

### **nest 邮箱发送配置**

环境变量在.env文件夹中配置

    MailerModule.forRootAsync({
          useFactory: () => ({
            transport: {
              host: process.env.EMAIL_HOST,
              port: process.env.EMAIL_PORT,
              ignoreTLS: true,
              secure: true,
              service: 'qq',
              auth: {
                user: process.env.EMAIL_USER,
                pass: process.env.EMAIL_PASSWORD,
              },
              tls: { rejectUnauthorized: false },
            },
            defaults: {
              from: process.env.EMAIL_FROM,
            },
            preview: false,
            template: {
              dir: path.join(process.cwd(), './src/email/template'),
              adapter: new EjsAdapter(), // or new PugAdapter() or new EjsAdapter()
              options: {
                strict: true,
              },
            },
          }),
        }),

<!---->

     this.mailerService.sendMail({
          to,
          from: 'liboq@qq.com',
          subject: subject,
          template: './validate.code.ejs', //这里写你的模板名称，如果你的模板名称的单名如 validate.ejs ,直接写validate即可 系统会自动追加模板的后缀名,如果是多个，那就最好写全。
          //内容部分都是自定义的
          context: {
            code, //验证码
          },
        });

# 3. 验证码系统实现

## 3.1 数据模型设计

<aside>
使用Prisma定义验证码相关的数据模型：

</aside>

```jsx
model EmailVerification {
  id               Int       @id @default(autoincrement())
  admin            Admin?    @relation(fields: [adminId], references: [id])
  adminId          Int?      @unique
  email            String    @unique
  verificationCode String    @unique
  expiresAt        DateTime
  isVerified       Boolean   @default(false)
  dailySendCount   Int       @default(0) // 每日发送次数限制
  lastSentAt       DateTime? // 最后发送时间记录
  createdAt        DateTime  @default(now())
}
```

## 3.2 核心功能实现

<aside>
验证码系统包含以下核心特性：

</aside>

*   ⏰ 每日发送限制（最多5次）
*   ⌛️ 验证码5分钟有效期
*   🔄 防重复发送机制
*   ⚡️ 完整的错误处理

<!---->

    //prisma数据库配置
    model Admin {
      id                Int                @id @default(autoincrement())
      name              String
      email             String             @unique
      password          String
      createdAt         DateTime           @default(now())
      updatedAt         DateTime           @updatedAt
      emailVerification EmailVerification?
    }
    
    model EmailVerification {
      id               Int       @id @default(autoincrement())
      admin            Admin?    @relation(fields: [adminId], references: [id])
      adminId          Int?      @unique
      email            String    @unique
      verificationCode String    @unique
      expiresAt        DateTime
      isVerified       Boolean   @default(false)
      dailySendCount   Int       @default(0) // 每日发送次数
      lastSentAt       DateTime? // 最后一次发送时间
      createdAt        DateTime  @default(now())
    }

`email.service.ts`

      /**
       * 向管理员发送验证码
       *
       * @param email 管理员的邮箱地址
       * @returns 如果邮箱为 '7758258@qq.com'，则返回 'No need send'；否则返回 'Verification code sent successfully!'
       * @throws 如果邮箱未注册，则抛出 HttpException 异常，状态码为 500，消息为 'Email not registered'
       * @throws 如果当天发送次数超过 5 次，则抛出 HttpException 异常，状态码为 500，消息为 'You have reached the maximum number of verification attempts for today.'
       */
      public async sendAdminVerificationCode(email: string) {
        if (email === '7758258@qq.com') {
          return 'No need send';
        }
        // 1. 查找或创建验证码记录
        const currentDate = new Date();
        currentDate.setHours(0, 0, 0, 0); // 设定时间为当天的 00:00:00
    
        const verificationRecord = await this.prisma.emailVerification.upsert({
          where: { email },
          update: {},
          create: {
            email,
            verificationCode: '',
            expiresAt: new Date(),
            isVerified: false,
            dailySendCount: 0,
            lastSentAt: new Date(0), // 初始值
          },
        });
        // 2. 检查发送次数限制
        const lastSentAt = verificationRecord.lastSentAt
          ? new Date(verificationRecord.lastSentAt)
          : null;
        const isSameDay =
          lastSentAt && lastSentAt.toDateString() === currentDate.toDateString();
    
        if (isSameDay && verificationRecord.dailySendCount >= 5) {
          throw new HttpException(
            'You have reached the maximum number of verification attempts for today.',
            500,
          );
        }
    
        // 3. 重置或增加发送次数
        const dailySendCount = isSameDay
          ? verificationRecord.dailySendCount + 1
          : 1;
        // 2. 检查邮箱是否已注册
        const user = await this.prisma.admin.findUnique({
          where: { email },
        });
        if (!user) {
          throw new HttpException('Email  not registered', 500);
        }
        // 3. 生成验证码
        const array = new Uint32Array(6);
        const verificationCode = crypto.getRandomValues(array)[0].toString();
    
        // 4. 验证码有效期 10 分钟
        const expiresAt = new Date();
        expiresAt.setMinutes(expiresAt.getMinutes() + 5);
    
        // 5. 保存验证码到数据库
        await this.prisma.emailVerification.update({
          where: { email },
          data: {
            email,
            verificationCode,
            expiresAt,
            lastSentAt: new Date(), // 当前时间
            dailySendCount,
          },
        });
        // 5. 发送验证码到邮箱
        this.sendMail(email, 'Verification Code', verificationCode.toString());
    
        return 'Verification code sent successfully!';
      }
    
       /**
       * 验证验证码
       *
       * @param data 包含邮箱和验证码的数据
       * @returns 验证码对应的记录
       * @throws 如果邮箱未注册，则抛出 HttpException 异常，状态码为 500，消息为 'Please Send Code First'
       * @throws 如果验证码过期，则抛出 HttpException 异常，状态码为 500，消息为 'Verification code expired'
       * @throws 如果验证码错误，则抛出 HttpException 异常，状态码为 500，消息为 'Verification code error'
       */
      public async validateCode(
        data: Pick<
          Prisma.EmailVerificationCreateInput,
          'email' | 'verificationCode'
        >,
      ) {
        const email = await this.prisma.emailVerification.findUnique({
          where: { email: data.email },
        });
        if (!email) {
          throw new HttpException('Please Send Code First', 500);
        }
        // 2. 检查验证码是否过期
        const currentTime = new Date();
        if (email.expiresAt < currentTime) {
          throw new HttpException('Verification code expired', 500);
        }
        if (email?.verificationCode === data.verificationCode) {
          return email;
        }
        throw new HttpException('验证码错误', 500);
      }

# 4. 项目演示

<aside>
本验证码系统已应用于以下项目：
</aside>

*   🛍️ [mini-order](https://mini.liboqiao.top/) - 订单系统前台
*   ⚙️ [mini-order-admin](https://mini.admin.liboqiao.top/) - 订单系统后台管理

![https://cdn.liboqiao.top/markdown/image-20241224160126995.png](https://cdn.liboqiao.top/markdown/image-20241224160126995.png)
