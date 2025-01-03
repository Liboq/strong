## NestJS 面试题

### 场景一：依赖注入和模块设计

```text
面试官：能谈谈你在 NestJS 项目中是如何使用依赖注入和设计模块的吗？

候选人：
好的,我来分享一下在实际项目中的经验。

首先是模块的设计。我们一般按照业务领域来划分模块,比如用户模块:
```

```typescript
// user.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User]), AuthModule, ConfigModule],
  controllers: [UserController],
  providers: [
    UserService,
    {
      provide: "USER_REPOSITORY",
      useClass: UserRepository,
    },
  ],
  exports: [UserService],
})
export class UserModule {}
```

```typescript
// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @Inject("USER_REPOSITORY")
    private userRepo: UserRepository,
    private authService: AuthService,
    private config: ConfigService
  ) {}

  async createUser(dto: CreateUserDto) {
    // 业务逻辑
  }
}
```

在依赖注入方面,我们遵循以下原则:

1. 单一职责:

   - 每个服务只负责一个领域的业务逻辑
   - 通过依赖注入组合功能

2. 接口分离:

   - 使用抽象接口定义契约
   - 具体实现通过 DI 容器注入

3. 依赖倒置:
   - 高层模块不依赖低层模块
   - 都依赖抽象接口

### 场景二：中间件和管道

````text
面试官：在处理请求时,你们是如何使用中间件和管道的？能举些实际的例子吗？

候选人：
好的,我们在项目中大量使用中间件和管道来处理通用逻辑。

1. 全局中间件处理跨域和日志:
```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.use(cors());
  app.use(helmet());
  app.use(morgan('combined'));

  await app.listen(3000);
}
````

2. 自定义验证管道:

```typescript
@Injectable()
export class ValidationPipe implements PipeTransform {
  async transform(value: any, metadata: ArgumentMetadata) {
    const { metatype } = metadata;
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }

    const object = plainToClass(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException("Validation failed");
    }
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

3. 请求响应拦截器:

```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => ({
        code: 0,
        data,
        message: "success",
      }))
    );
  }
}
```

这些工具的使用原则:

1. 中间件:

   - 处理通用的 HTTP 相关逻辑
   - 尽早拦截无效请求
   - 添加通用的请求信息

2. 管道:

   - 数据验证和转换
   - 参数类型转换
   - 自定义验证规则

3. 拦截器:
   - 统一的响应格式
   - 异常处理
   - 缓存处理

### 场景三：异常处理和日志记录

````text
面试官：在 NestJS 项目中，你们是如何处理异常和进行日志记录的？

候选人：
好的，我来分享一下我们的异常处理和日志记录方案。

1. 全局异常过滤器：
```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      message = exception.message;
    } else if (exception instanceof QueryFailedError) {
      status = HttpStatus.BAD_REQUEST;
      message = 'Database query failed';
    }

    // 记录错误日志
    this.logger.error(message, {
      exception,
      path: request.url,
      method: request.method,
      timestamp: new Date().toISOString()
    });

    response.status(status).json({
      code: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message
    });
  }
}
````

2. 自定义业务异常：

```typescript
export class BusinessException extends HttpException {
  constructor(
    message: string,
    code: number = HttpStatus.BAD_REQUEST,
    data?: any
  ) {
    super(
      {
        message,
        code,
        data
      },
      code
    );
  }
}

// 使用示例
@Get(':id')
async findOne(@Param('id') id: string) {
  const user = await this.userService.findOne(id);
  if (!user) {
    throw new BusinessException(
      'User not found',
      ErrorCode.USER_NOT_FOUND
    );
  }
  return user;
}
```

3. 日志服务封装：

```typescript
@Injectable()
export class LoggerService extends Logger {
  constructor(private config: ConfigService) {
    super();
  }

  log(message: string, context?: string) {
    if (this.config.get("env") !== "test") {
      super.log(message, context);
    }
  }

  error(message: string, trace?: string, context?: string) {
    // 发送错误通知
    this.notifyError(message, trace);
    super.error(message, trace, context);
  }

  private async notifyError(message: string, trace?: string) {
    // 发送到监控系统或告警服务
    await this.sendToMonitoring({
      message,
      trace,
      timestamp: new Date(),
      environment: this.config.get("env"),
    });
  }
}
```

4. 请求日志中间件：

```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  constructor(private readonly logger: LoggerService) {}

  use(req: Request, res: Response, next: NextFunction) {
    const startTime = Date.now();

    // 请求结束后记录日志
    res.on("finish", () => {
      const duration = Date.now() - startTime;
      this.logger.log(
        `${req.method} ${req.originalUrl} ${res.statusCode} ${duration}ms`
      );
    });

    next();
  }
}
```

我们的异常处理和日志记录策略包括：

1. 异常分类：

   - HTTP 异常
   - 业务异常
   - 数据库异常
   - 系统异常

2. 错误处理：

   - 统一的错误响应格式
   - 详细的错误信息记录
   - 错误通知机制

3. 日志分级：

   - DEBUG：调试信息
   - INFO：一般信息
   - WARN：警告信息
   - ERROR：错误信息

4. 日志管理：
   - 日志分类存储
   - 日志轮转策略
   - 日志查询工具

### 场景四：数据库设计和事务处理

````text
面试官：在 NestJS 项目中，你们是如何处理数据库操作和事务的？能分享一些实践经验吗？

候选人：
好的，我来分享一下我们在数据库操作方面的实践。

1. 实体设计和关联：
```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column({ select: false })
  password: string;

  @OneToMany(() => Order, order => order.user)
  orders: Order[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @BeforeInsert()
  async hashPassword() {
    if (this.password) {
      this.password = await bcrypt.hash(this.password, 10);
    }
  }
}
````

2. 事务处理：

```typescript
@Injectable()
export class OrderService {
  constructor(
    @InjectRepository(Order)
    private orderRepo: Repository<Order>,
    @InjectRepository(Product)
    private productRepo: Repository<Product>,
    private connection: Connection
  ) {}

  async createOrder(dto: CreateOrderDto) {
    // 使用查询运行器处理事务
    const queryRunner = this.connection.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      // 检查库存
      const product = await queryRunner.manager.findOne(Product, dto.productId);
      if (product.stock < dto.quantity) {
        throw new BusinessException("Insufficient stock");
      }

      // 创建订单
      const order = queryRunner.manager.create(Order, {
        ...dto,
        totalAmount: product.price * dto.quantity,
      });
      await queryRunner.manager.save(order);

      // 更新库存
      product.stock -= dto.quantity;
      await queryRunner.manager.save(product);

      await queryRunner.commitTransaction();
      return order;
    } catch (err) {
      await queryRunner.rollbackTransaction();
      throw err;
    } finally {
      await queryRunner.release();
    }
  }
}
```

3. 数据库查询优化：

```typescript
@Injectable()
export class ProductService {
  constructor(
    @InjectRepository(Product)
    private productRepo: Repository<Product>
  ) {}

  async findProducts(query: ProductQueryDto) {
    // 构建查询构建器
    const qb = this.productRepo
      .createQueryBuilder("product")
      .leftJoinAndSelect("product.category", "category")
      .where("product.isActive = :isActive", { isActive: true });

    // 条件查询
    if (query.category) {
      qb.andWhere("category.id = :categoryId", {
        categoryId: query.category,
      });
    }

    if (query.minPrice) {
      qb.andWhere("product.price >= :minPrice", {
        minPrice: query.minPrice,
      });
    }

    // 排序和分页
    qb.orderBy("product.createdAt", "DESC")
      .skip((query.page - 1) * query.limit)
      .take(query.limit);

    // 执行查询
    const [items, total] = await qb.getManyAndCount();

    return {
      items,
      total,
      page: query.page,
      limit: query.limit,
    };
  }
}
```

4. 数据库索引和性能优化：

```typescript
@Entity("products")
@Index(["name", "category"])
export class Product {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Index()
  @Column()
  name: string;

  @ManyToOne(() => Category)
  @Index()
  category: Category;

  @Column("decimal", { precision: 10, scale: 2 })
  price: number;

  @Column("int")
  stock: number;

  @Index()
  @CreateDateColumn()
  createdAt: Date;
}
```

我们的数据库最佳实践包括：

1. 实体设计：

   - 合理的字段类型
   - 必要的字段索引
   - 关联关系设计

2. 事务处理：

   - 合理的事务边界
   - 正确的错误处理
   - 事务隔离级别

3. 查询优化：

   - 使用查询构建器
   - 避免 N+1 问题
   - 合理使用缓存

4. 性能监控：
   - SQL 日志记录
   - 慢查询分析
   - 性能指标监控

### 场景五：微服务和消息队列

````text
面试官：你们在 NestJS 项目中是如何实现微服务架构和处理消息队列的？

候选人：
好的，我来分享一下我们在微服务和消息队列方面的实践经验。

1. 微服务通信：
```typescript
// 用户服务
@Controller()
export class UserController {
  @MessagePattern({ cmd: 'get_user' })
  async getUser(@Payload() id: string) {
    return this.userService.findOne(id);
  }

  @EventPattern('user_created')
  async handleUserCreated(@Payload() data: any) {
    // 处理用户创建事件
    await this.userService.handleUserCreated(data);
  }
}

// 订单服务
@Injectable()
export class OrderService {
  constructor(
    @Inject('USER_SERVICE') private userClient: ClientProxy
  ) {}

  async createOrder(userId: string, dto: CreateOrderDto) {
    // 验证用户
    const user = await firstValueFrom(
      this.userClient.send({ cmd: 'get_user' }, userId)
    );

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // 创建订单
    const order = await this.orderRepo.save({
      ...dto,
      userId
    });

    // 发送订单创建事件
    this.userClient.emit('order_created', order);

    return order;
  }
}
````

2. 消息队列集成：

```typescript
// 消息队列配置
@Module({
  imports: [
    ClientsModule.register([
      {
        name: "ORDER_SERVICE",
        transport: Transport.RMQ,
        options: {
          urls: ["amqp://localhost:5672"],
          queue: "order_queue",
          queueOptions: {
            durable: true,
          },
        },
      },
    ]),
  ],
})
export class OrderModule {}

// 消息生产者
@Injectable()
export class OrderProducer {
  constructor(
    @Inject("ORDER_SERVICE")
    private client: ClientProxy
  ) {}

  async sendOrderCreatedEvent(order: Order) {
    try {
      await this.client.emit("order_created", {
        id: order.id,
        userId: order.userId,
        amount: order.amount,
        timestamp: new Date(),
      });
    } catch (error) {
      // 处理消息发送失败
      this.logger.error("Failed to send order created event", error);
    }
  }
}
```

3. 消息消费者：

```typescript
@Controller()
export class NotificationConsumer {
  @EventPattern("order_created")
  async handleOrderCreated(@Payload() data: any) {
    try {
      // 发送通知
      await this.notificationService.sendOrderNotification(data);
    } catch (error) {
      // 处理失败重试
      this.handleFailedNotification(data, error);
    }
  }

  private async handleFailedNotification(data: any, error: any) {
    // 记录失败日志
    this.logger.error("Failed to process notification", {
      data,
      error,
    });

    // 重试队列
    await this.retryQueue.add("send_notification", {
      data,
      attempt: 1,
    });
  }
}
```

4. 服务发现和负载均衡：

```typescript
// 服务注册
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.TCP,
    options: {
      host: 'localhost',
      port: 3001,
    },
  },
);

// 服务发现配置
@Module({
  imports: [
    ClientsModule.registerAsync([
      {
        name: 'USER_SERVICE',
        useFactory: (configService: ConfigService) => ({
          transport: Transport.TCP,
          options: {
            host: configService.get('USER_SERVICE_HOST'),
            port: configService.get('USER_SERVICE_PORT'),
          },
        }),
        inject: [ConfigService],
      },
    ]),
  ],
})
```

在微服务架构中，我们遵循以下原则：

1. 服务设计：

   - 单一职责原则
   - 服务边界清晰
   - 数据独立性

2. 通信模式：

   - 同步通信 (RPC)
   - 异步事件
   - 消息持久化

3. 可靠性保证：

   - 消息重试机制
   - 死信队列处理
   - 幂等性设计

4. 监控和追踪：
   - 分布式追踪
   - 服务健康检查
   - 性能监控

### 场景六：NestJS vs Koa

````text
面试官：能谈谈 NestJS 和 Koa 的主要区别吗？为什么选择 NestJS？

候选人：
好的，我可以从几个方面来比较。我们团队之前用过 Koa，后来迁移到了 NestJS，有一些切身体会。

1. 架构设计：
```typescript
// Koa 的中间件模式
const Koa = require('koa');
const app = new Koa();

app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});

// NestJS 的模块化设计
@Module({
  imports: [
    TypeOrmModule.forRoot(),
    UserModule,
    AuthModule
  ],
  controllers: [AppController],
  providers: [AppService]
})
export class AppModule {}
````

主要区别：

1. 架构风格：

   - Koa：轻量级、中间件驱动
   - NestJS：完整框架、模块化设计

2. 开发模式：

   - Koa：自由度高，需要自己组织代码结构
   - NestJS：约定优于配置，有清晰的项目结构

3. 功能特性：

   ```typescript
   // Koa 需要自己集成功能
   const Router = require("koa-router");
   const bodyParser = require("koa-bodyparser");
   const cors = require("@koa/cors");

   app.use(bodyParser());
   app.use(cors());

   // NestJS 内置了很多功能
   @Controller("users")
   export class UserController {
     @Post()
     @UsePipes(ValidationPipe)
     @UseGuards(AuthGuard)
     async createUser(@Body() dto: CreateUserDto) {
       return this.userService.create(dto);
     }
   }
   ```

4. 类型支持：

   ```typescript
   // Koa 需要额外配置类型
   interface State {
     user: {
       id: string;
       name: string;
     };
   }

   interface Context extends Koa.Context {
     state: State;
   }

   // NestJS 原生支持 TypeScript
   @Injectable()
   export class UserService {
     async findOne(id: string): Promise<User> {
       return this.userRepo.findOne(id);
     }
   }
   ```

选择 NestJS 的原因：

1. 企业级开发：

   - 更好的代码组织
   - 更强的可维护性
   - 完整的技术生态

2. 开发效率：

   - 开箱即用的功能
   - 清晰的项目结构
   - 完善的文档支持

3. 团队协作：

   - 统一的开发规范
   - 更容易的知识共享
   - 更好的可扩展性

4. 性能和可靠性：
   - 依赖注入带来的松耦合
   - 装饰器提供的元编程能力
   - 内置的性能优化

### 场景七：洋葱模型的实现与应用

````text
面试官：能谈谈你对洋葱模型的理解，以及在 NestJS 中是如何应用的吗？

候选人：
好的，我来分享一下对洋葱模型的理解和实践经验。

1. 洋葱模型的基本原理：
```typescript
// Koa 中的洋葱模型实现
app.use(async (ctx, next) => {
  console.log('1. 请求进入第一层中间件');
  await next();
  console.log('6. 响应离开第一层中间件');
});

app.use(async (ctx, next) => {
  console.log('2. 请求进入第二层中间件');
  await next();
  console.log('5. 响应离开第二层中间件');
});

app.use(async (ctx, next) => {
  console.log('3. 请求进入第三层中间件');
  ctx.body = 'Hello World';
  console.log('4. 响应离开第三层中间件');
});

// 执行顺序：1 -> 2 -> 3 -> 4 -> 5 -> 6
````

2. NestJS 中的应用：

```typescript
// 中间件
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    console.log("1. 请求进入 Logger 中间件");
    await next();
    console.log("6. 响应离开 Logger 中间件");
  }
}

// 守卫
@Injectable()
export class AuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext) {
    console.log("2. 请求进入 Auth 守卫");
    // 执行认证逻辑
    console.log("5. 响应离开 Auth 守卫");
    return true;
  }
}

// 拦截器
@Injectable()
export class TimeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log("3. 请求进入拦截器");
    return next.handle().pipe(tap(() => console.log("4. 响应离开拦截器")));
  }
}
```

3. 实际应用场景：

```typescript
@Controller("users")
@UseGuards(AuthGuard)
@UseInterceptors(TimeInterceptor)
export class UserController {
  @Post()
  async createUser(@Body() dto: CreateUserDto) {
    // 处理请求
    const startTime = Date.now();

    try {
      // 业务逻辑
      const result = await this.userService.create(dto);

      // 记录执行时间
      const duration = Date.now() - startTime;
      this.logger.log(`创建用户耗时: ${duration}ms`);

      return result;
    } catch (error) {
      // 错误会沿着洋葱模型向外传播
      throw error;
    }
  }
}
```

洋葱模型的优势：

1. 请求处理流程：

   - 可以清晰地控制请求的处理流程
   - 中间件按照洋葱层次依次执行
   - 响应按照相反顺序处理

2. 错误处理：

   - 错误可以在任何层次被捕获
   - 统一的错误处理机制
   - 优雅的错误传播

3. 代码组织：

   - 关注点分离
   - 逻辑复用
   - 易于维护和测试

4. 性能监控：
   - 方便添加性能监控逻辑
   - 精确的执行时间统计
   - 请求链路追踪

### 场景八：中间件的分类和使用

````text
面试官：能详细说说 NestJS 中间件的分类和具体使用场景吗？

候选人：
好的，我来分享一下中间件的不同类型和实际应用场景。

1. 功能中间件：
```typescript
// 请求日志中间件
@Injectable()
export class RequestLoggerMiddleware implements NestMiddleware {
  constructor(private logger: LoggerService) {}

  use(req: Request, res: Response, next: NextFunction) {
    const { method, originalUrl, ip } = req;
    this.logger.log(`${method} ${originalUrl} - ${ip}`);
    next();
  }
}

// 在模块中应用
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(RequestLoggerMiddleware)
      .forRoutes('*');
  }
}
````

2. 函数式中间件：

```typescript
// CORS 中间件
export function corsMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Methods", "GET,PUT,POST,DELETE");
  res.header("Access-Control-Allow-Headers", "Content-Type, Accept");

  if (req.method === "OPTIONS") {
    res.sendStatus(200);
  } else {
    next();
  }
}

// 在模块中应用
consumer.apply(corsMiddleware).forRoutes("*");
```

3. 全局中间件：

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 全局中间件
  app.use(helmet());
  app.use(compression());
  app.use(morgan("combined"));

  await app.listen(3000);
}
```

4. 路由中间件：

```typescript
// 权限检查中间件
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(private authService: AuthService) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers["authorization"];

    if (!token) {
      throw new UnauthorizedException("No token provided");
    }

    try {
      const user = await this.authService.verifyToken(token);
      req["user"] = user;
      next();
    } catch (error) {
      throw new UnauthorizedException("Invalid token");
    }
  }
}

// 在模块中应用到特定路由
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(AuthMiddleware)
      .exclude(
        { path: "auth/login", method: RequestMethod.POST },
        { path: "auth/register", method: RequestMethod.POST }
      )
      .forRoutes(
        { path: "users/*", method: RequestMethod.ALL },
        { path: "orders/*", method: RequestMethod.ALL }
      );
  }
}
```

中间件的使用场景：

1. 请求预处理：

   - 请求日志记录
   - 请求参数转换
   - 请求头处理

2. 安全相关：

   - CORS 处理
   - 安全头设置
   - XSS 防护

3. 认证授权：

   - Token 验证
   - 权限检查
   - 角色验证

4. 性能优化：
   - 响应压缩
   - 缓存控制
   - 限流处理

中间件的最佳实践：

1. 职责划分：

   - 每个中间件只负责一个功能
   - 合理组合多个中间件
   - 保持中间件的独立性

2. 执行顺序：

   - 全局中间件最先执行
   - 模块中间件其次执行
   - 路由中间件最后执行

3. 性能考虑：
   - 避免重复执行
   - 合理使用异步操作
   - 注意内存使用
