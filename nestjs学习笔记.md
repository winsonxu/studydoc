# 安装 cil 工具
npm i -g @nestjs/cli

# 创建新项目
nest new project-name

# 开启跨站请求
main.js 中增加
app.enableCors();

# 设置接口路径
app.setGlobalPerfix('api')

# 包装异常返回对象和正常返回对象
nest g f http-exception --no-spec
nest g in transform --no-spec

main.js 中增加
app.useGlobalFilters(new HttpExceptionFilter());
app.useGlobalInterceptors(new TransformInterceptor());

# 安装 swgger 自动生成 api 文档
npm i -D @nestjs/swagger swagger-ui-express

main.js 中增加
const options = new DocumentBuilder().setTitle('商业地产 API').setDescription('V1版本的 API').setVersion('1.0').build();
const document = SwaggerModule.createDocument(app, options);
SwaggerModule.setup('api-doc', app, document);

# 安装 type-orm
npm i -D @nestjs/typeorm typeorm mysql
根目录添加 ormconfig.json
{
    "type":"mysql",
    "host":"localhost",
    "port": 3306,
    "username":"root",
    "password":"root",
    "database":"office_norait_cn",
    "entities": ["dist/**/*.entity{.ts,.js}"],
    "synchronize": true,
    "logging": true
}

app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot(),
  ],
})

整个项目中注入Connection和 EntityManager
export class AppModule {
  constructor(private connection: Connection) {}
}

创建模块、服务、控制器
nest g mo Hreo --no-spec && nest g s Hreo --no-spec &&nest g co Hreo --no-spec


#安装 CRUD 自动生成增删改查
npm i -D @nestjsx/crud @nestjsx/crud-typeorm class-transformer class-validator
创建 user 模块
nest g mo user --no-spec
创建 user.entity.ts 文件
@Entity('tb_user')
export class User{
    @PrimaryGeneratedColumn()
    id: number;
    @Column({ comment: '创建时间' })
    username: string;
    @CreateDateColumn({ comment: '创建时间' })  // 自动生成列
    createdAt: string
    @UpdateDateColumn({ comment: '更新时间' })   // 自动生成并自动更新列
    updatedAt: string
...
}
在 user.model 中导入 UserEntity
@Module({
  imports:[TypeOrmModule.forFeature([User])],

创建 user.service
nest g s user --no-spec
@Injectable()
export class UserService extends TypeOrmCrudService<User> {
    constructor(@InjectRepository(User) repo) { super(repo); }
}
创建 controller
nest g co user --no-spec

@Crud({model: { type: User} })
这里可以选择只开哪些自动生成功能：,routes:{ only:[ 'getManyBase' ] }

@Controller('user')
export class HeroesController {
  constructor(public service: UserService) {}
}


#增加API 请求 token 验证
npm install --save @nestjs/passport passport passport-jwt
npm install --save-dev @types/passport-local


#VSCode 排除不用的文件显示
设置CMD+SHIFT+P  SETTING  WORKSPACE-SETTING
{
    "files.exclude": {
        "**/*.spec.ts": true,
        "**/dist": true,
        "**/node_modules": true
    },
    "search.exclude": {
        "**/dist": true
    }
}

#Swagger配置装饰器https://docs.nestjs.com/recipes/swagger
@ApiTags('用户')
@ApiBody
@ApiHeaders
@ApiParam({
        name: 'id',
        description: '这是用户id',
    })
@ApiQuery({
        name: 'role',
        description: '这是需要传递的参数',
    })
@ApiHeader({
        name: 'authoriation',
        required: true,
        description: '本次请求请带上token',
    })
@ApiProperty({
        description: '用户名',
    })
@ApiResponse({ status: 401, description: '权限不足'})
@ApiImplicitFile({
        name: '头像',
        description: '上传头像',
        required: false,
    })



#添加JWT验证

npm i -D @nestjs/jwt  passport-jwt @nestjs/passport passport @types/passport-jwt --registory=https://registory.npm.taobao.org

nest g mo auth --no-spec && nest g s auth --no-spec && nest g co auth --no-spec 

创建jwt密钥和jwtpayload
export const jwtConstants = { secret: 'f87727dfecc28725105a769dc4fe4c8f'}
export class JwtPayload { username: string; id: number; }

创建策略JwtStrategy
在Auth.module.ts中引入
在auth.service.ts中增加验证、登陆函数
创建一个装饰器
nest g d user --no-spec
在Controller中增加 @UseGuards(AuthGuard('jwt'))@ApiBearerAuth('token')
在Controller的action中@User来获取
在main.ts中增加.addBearerAuth({name:'token',type:'http'},'token')



发布：
前端用npm run build来生成一个dist然后放到根目录下
node项目需要将源dist和根目录下除src和test外其它文件也一起打包放到服务器上
配置服务器上的数据库连接字符串
node需要在服务器上npm run build，然后pm2配置启动为dist/main.js
nginx需要配置反向代理
    location /api/
    {
        proxy_pass http://crm.norait.cn:6543;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header REMOTE-HOST $remote_addr;
    }

当类之间具有循环依赖关系时，请使用惰性函数为 SwaggerModule 提供类型信息：
@ApiProperty({ type: () => Node })


nodejs post数据大小限制
nodejs Error: request entity too large解决方案
npm i body-parser
var bodyParser = require('body-parser');
app.use(bodyParser.json({limit: '50mb'}));
app.use(bodyParser.urlencoded({limit: '50mb', extended: true}));

#使用配置文件.env
npm i --save @nestjs/config


#数据迁移
ts-node ./node_modules/typeorm/cli.js migration:run


