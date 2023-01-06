**女主宣言**

今天小编为大家分享基于DDD的golang实现，DDD即领域驱动设计，该模式也算是比较热门的话题了。希望通过本篇文章，大家能够掌握DDD模式，能对大家有所帮助。

**PS：丰富的一线技术、多元化的表现形式，尽在“360云计算”，点关注哦！**

领域驱动设计模式算是比较热门的话题了。

领域驱动设计（DDD）是一种软件开发方法，通过将实现与不断演变的模型相连接，简化了开发人员面临的复杂性。

本文不会重点去解释Golang中实现DDD的相关理念，而是作者根据自己的研究对DDD的理解。 

什么是DDD？

以下是考虑使用DDD的原因：

-   提供解决困难问题的原则和模式
    
-   将复杂的设计基于领域模型
    
-   在技术和领域专家之间发起创造性的协作，以迭代地完善解决领域问题的概念模型。
    

  
DDD包含4个层：

1.  Domain：这是定义应用程序的域和业务逻辑的地方
    
2.  Infrastructure：此层包含独立于我们的应用程序而存在的所有内容：外部库，数据库引擎等。
    
3.  Application：该层用作域和界面层之间的通道。将请求从接口层发送到域层，由域层处理请求并返回响应。
    
4.  Interface：该层包含与其他系统交互的所有内容，例如Web服务，RMI接口或Web应用程序以及批处理前端。
    

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6MZqUI3KNlrmvFy8zdkpwcnd8HfdeLMBhW6tcM5V9hulUA2wRHiaNT1qjPrtot92phcquv3ZloQiaWwcgDPtv9Nw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1

**开始**

我们将构建一个食物推荐API。

首先要做的是初始化依赖关系管理。我们将使用go.mod。在根目录（路径：food-app /）中，初始化go.mod：

```
go mod init food-app
```

项目的组织结构：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6MZqUI3KNlrmvFy8zdkpwcnd8HfdeLMBibeSt7XNGFeJ2v8TlLwVLRIq9XjvcBXur9vzhMlptYEF9Rt9v4cgNibg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在该应用中，我们将使用postgres和redis数据库持久化数据。先定义一个含有连接信息的.env文件。  
.env文件内容：

```
#PostgresAPP_ENV=localAPI_PORT=8888DB_HOST=127.0.0.1DB_DRIVER=postgresACCESS_SECRET=98hbun98hREFRESH_SECRET=786dfdbjhsbDB_USER=stevenDB_PASSWORD=passwordDB_NAME=food-appDB_PORT=5432#Mysql#DB_HOST=127.0.0.1#DB_DRIVER=mysql#DB_USER=steven#DB_PASSWORD=here#DB_NAME=food-app#DB_PORT=3306#Postgres Test DBTEST_DB_DRIVER=postgresTEST_DB_HOST=127.0.0.1TEST_DB_PASSWORD=passwordTEST_DB_USER=stevenTEST_DB_NAME=food-app-testTEST_DB_PORT=5432#RedisREDIS_HOST=127.0.0.1REDIS_PORT=6379REDIS_PASSWORD=
```

该文件应位于根目录中（路径：food-app /）

2

**Domain 层**

我们将首先考虑领域。

该域具有几种模式。其中一些是：实体，值，存储库，服务等。

由于我们在此处构建的应用比较简单，因此我们仅考虑两种域模式：实体和存储库。

**实体**

这是我们定义“Schema”的地方。  
例如，我们可以定义用户的结构。将该实体视为域的蓝图。

```
package entityimport ("food-app/infrastructure/security""github.com/badoux/checkmail""html""strings""time")type User struct {ID        uint64     `gorm:"primary_key;auto_increment" json:"id"`FirstName string     `gorm:"size:100;not null;" json:"first_name"`LastName  string     `gorm:"size:100;not null;" json:"last_name"`Email     string     `gorm:"size:100;not null;unique" json:"email"`Password  string     `gorm:"size:100;not null;" json:"password"`CreatedAt time.Time  `gorm:"default:CURRENT_TIMESTAMP" json:"created_at"`UpdatedAt time.Time  `gorm:"default:CURRENT_TIMESTAMP" json:"updated_at"`DeletedAt *time.Time `json:"deleted_at,omitempty"`}type PublicUser struct {ID        uint64 `gorm:"primary_key;auto_increment" json:"id"`FirstName string `gorm:"size:100;not null;" json:"first_name"`LastName  string `gorm:"size:100;not null;" json:"last_name"`}//BeforeSave is a gorm hookfunc (u *User) BeforeSave() error {hashPassword, err := security.Hash(u.Password)if err != nil {return err}u.Password = string(hashPassword)return nil}type Users []User//So that we dont expose the user's email address and password to the worldfunc (users Users) PublicUsers() []interface{} {result := make([]interface{}, len(users))for index, user := range users {result[index] = user.PublicUser()}return result}//So that we dont expose the user's email address and password to the worldfunc (u *User) PublicUser() interface{} {return &PublicUser{ID:        u.ID,FirstName: u.FirstName,LastName:  u.LastName,}}func (u *User) Prepare() {u.FirstName = html.EscapeString(strings.TrimSpace(u.FirstName))u.LastName = html.EscapeString(strings.TrimSpace(u.LastName))u.Email = html.EscapeString(strings.TrimSpace(u.Email))u.CreatedAt = time.Now()u.UpdatedAt = time.Now()}func (u *User) Validate(action string) map[string]string {var errorMessages = make(map[string]string)var err errorswitch strings.ToLower(action) {case "update":if u.Email == "" {errorMessages["email_required"] = "email required"}if u.Email != "" {if err = checkmail.ValidateFormat(u.Email); err != nil {errorMessages["invalid_email"] = "email email"}}case "login":if u.Password == "" {errorMessages["password_required"] = "password is required"}if u.Email == "" {errorMessages["email_required"] = "email is required"}if u.Email != "" {if err = checkmail.ValidateFormat(u.Email); err != nil {errorMessages["invalid_email"] = "please provide a valid email"}}case "forgotpassword":if u.Email == "" {errorMessages["email_required"] = "email required"}if u.Email != "" {if err = checkmail.ValidateFormat(u.Email); err != nil {errorMessages["invalid_email"] = "please provide a valid email"}}default:if u.FirstName == "" {errorMessages["firstname_required"] = "first name is required"}if u.LastName == "" {errorMessages["lastname_required"] = "last name is required"}if u.Password == "" {errorMessages["password_required"] = "password is required"}if u.Password != "" && len(u.Password) < 6 {errorMessages["invalid_password"] = "password should be at least 6 characters"}if u.Email == "" {errorMessages["email_required"] = "email is required"}if u.Email != "" {if err = checkmail.ValidateFormat(u.Email); err != nil {errorMessages["invalid_email"] = "please provide a valid email"}}}return errorMessages}
```

在上面的文件中，定义了包含用户信息的用户结构，我们还添加了帮助程序功能，这些功能将验证和清理输入。调用了一种哈希方法，该方法用于哈希密码。这是在基础结构层中定义的。  
定义 food 实体时采用相同的方法。

**存储库**

存储库定义了基础结构实现的方法的集合。这描绘了与给定数据库或第三方API交互的方法数量。

user 存储库如下所示：

```
package repositoryimport ("food-app/domain/entity")type UserRepository interface {SaveUser(*entity.User) (*entity.User, map[string]string)GetUser(uint64) (*entity.User, error)GetUsers() ([]entity.User, error)GetUserByEmailAndPassword(*entity.User) (*entity.User, map[string]string)}
```

方法在接口中定义。这些方法稍后将在基础结构层中实现。

food 库几乎相同。

3

**Infrastructure 层**

该层实现存储库中定义的方法。这些方法与数据库或第三方API交互。本文中仅考虑数据库交互。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6MZqUI3KNlrmvFy8zdkpwcnd8HfdeLMBJmhs39SPFdDBRvGltnChaMicnJoRHrw012AzdlRtBtfRhBKgvdfblPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以看到 user 存储库实现如下所示：

```
package persistenceimport ("errors""food-app/domain/entity""food-app/domain/repository""food-app/infrastructure/security""github.com/jinzhu/gorm""golang.org/x/crypto/bcrypt""strings")type UserRepo struct {db *gorm.DB}func NewUserRepository(db *gorm.DB) *UserRepo {return &UserRepo{db}}//UserRepo implements the repository.UserRepository interfacevar _ repository.UserRepository = &UserRepo{}func (r *UserRepo) SaveUser(user *entity.User) (*entity.User, map[string]string) {dbErr := map[string]string{}err := r.db.Debug().Create(&user).Errorif err != nil {//If the email is already takenif strings.Contains(err.Error(), "duplicate") || strings.Contains(err.Error(), "Duplicate") {dbErr["email_taken"] = "email already taken"return nil, dbErr}//any other db errordbErr["db_error"] = "database error"return nil, dbErr}return user, nil}func (r *UserRepo) GetUser(id uint64) (*entity.User, error) {var user entity.Usererr := r.db.Debug().Where("id = ?", id).Take(&user).Errorif err != nil {return nil, err}if gorm.IsRecordNotFoundError(err) {return nil, errors.New("user not found")}return &user, nil}func (r *UserRepo) GetUsers() ([]entity.User, error) {var users []entity.Usererr := r.db.Debug().Find(&users).Errorif err != nil {return nil, err}if gorm.IsRecordNotFoundError(err) {return nil, errors.New("user not found")}return users, nil}func (r *UserRepo) GetUserByEmailAndPassword(u *entity.User) (*entity.User, map[string]string) {var user entity.UserdbErr := map[string]string{}err := r.db.Debug().Where("email = ?", u.Email).Take(&user).Errorif gorm.IsRecordNotFoundError(err) {dbErr["no_user"] = "user not found"return nil, dbErr}if err != nil {dbErr["db_error"] = "database error"return nil, dbErr}//Verify the passworderr = security.VerifyPassword(user.Password, u.Password)if err != nil && err == bcrypt.ErrMismatchedHashAndPassword {dbErr["incorrect_password"] = "incorrect password"return nil, dbErr}return &user, nil}
```

可以看到我们实现了存储库中定义的方法。使用实现了UserRepository接口的UserRepo结构可以做到这一点，如下行所示：

```
//UserRepo implements the repository.UserRepository interfacevar _ repository.UserRepository = &UserRepo{}
```

因此，我们通过创建包含以下内容的db.go文件来配置数据库：

```
package persistenceimport ("fmt""food-app/domain/entity""food-app/domain/repository""github.com/jinzhu/gorm"_ "github.com/jinzhu/gorm/dialects/postgres")type Repositories struct {User repository.UserRepositoryFood repository.FoodRepositorydb   *gorm.DB}func NewRepositories(Dbdriver, DbUser, DbPassword, DbPort, DbHost, DbName string) (*Repositories, error) {DBURL := fmt.Sprintf("host=%s port=%s user=%s dbname=%s sslmode=disable password=%s", DbHost, DbPort, DbUser, DbName, DbPassword)db, err := gorm.Open(Dbdriver, DBURL)if err != nil {return nil, err}db.LogMode(true)return &Repositories{User: NewUserRepository(db),Food: NewFoodRepository(db),db:   db,}, nil}//closes the  database connectionfunc (s *Repositories) Close() error {return s.db.Close()}//This migrate all tablesfunc (s *Repositories) Automigrate() error {return s.db.AutoMigrate(&entity.User{}, &entity.Food{}).Error}
```

在上面的文件中，我们定义了Repositories结构，该结构保存了应用中的所有存储库。我们有 user 和 food 库。该存储库还具有一个db实例，该实例被传递给 user 和 food（即NewUserRepository和NewFoodRepository）的“constructors”。

4

**Application 层**

我们已经在域中定义了API业务逻辑。该层连接 domain 和 interfaces 层。

以下是 user 的应用层：

```
package applicationimport ("food-app/domain/entity""food-app/domain/repository")type userApp struct {us repository.UserRepository}//UserApp implements the UserAppInterfacevar _ UserAppInterface = &userApp{}type UserAppInterface interface {SaveUser(*entity.User) (*entity.User, map[string]string)GetUsers() ([]entity.User, error)GetUser(uint64) (*entity.User, error)GetUserByEmailAndPassword(*entity.User) (*entity.User, map[string]string)}func (u *userApp) SaveUser(user *entity.User) (*entity.User, map[string]string) {return u.us.SaveUser(user)}func (u *userApp) GetUser(userId uint64) (*entity.User, error) {return u.us.GetUser(userId)}func (u *userApp) GetUsers() ([]entity.User, error) {return u.us.GetUsers()}func (u *userApp) GetUserByEmailAndPassword(user *entity.User) (*entity.User, map[string]string) {return u.us.GetUserByEmailAndPassword(user)}
```

上面有保存和检索用户数据的方法。UserApp结构具有UserRepository接口，从而可以调用用户存储库方法。

5

**Interfaces 层**

接口是处理HTTP请求和响应的层。这里我们收到身份验证，与用户相关的内容和与食品相关的内容的传入请求。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6MZqUI3KNlrmvFy8zdkpwcnd8HfdeLMB0Lic96zJiclIuJ3LHsQDv2abNLNz9Mm2P51vL2tursCOWBXH36ggaUaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**用户处理**

我们定义了保存用户，获取所有用户和获取特定用户的方法。这些可以在user\_handler.go文件中找到。

```
package interfacesimport ("food-app/application""food-app/domain/entity""food-app/infrastructure/auth""github.com/gin-gonic/gin""net/http""strconv")//Users struct defines the dependencies that will be usedtype Users struct {us application.UserAppInterfacerd auth.AuthInterfacetk auth.TokenInterface}//Users constructorfunc NewUsers(us application.UserAppInterface, rd auth.AuthInterface, tk auth.TokenInterface) *Users {return &Users{us: us,rd: rd,tk: tk,}}func (s *Users) SaveUser(c *gin.Context) {var user entity.Userif err := c.ShouldBindJSON(&user); err != nil {c.JSON(http.StatusUnprocessableEntity, gin.H{"invalid_json": "invalid json",})return}//validate the request:validateErr := user.Validate("")if len(validateErr) > 0 {c.JSON(http.StatusUnprocessableEntity, validateErr)return}newUser, err := s.us.SaveUser(&user)if err != nil {c.JSON(http.StatusInternalServerError, err)return}c.JSON(http.StatusCreated, newUser.PublicUser())}func (s *Users) GetUsers(c *gin.Context) {users := entity.Users{} //customize uservar err error//us, err = application.UserApp.GetUsers()users, err = s.us.GetUsers()if err != nil {c.JSON(http.StatusInternalServerError, err.Error())return}c.JSON(http.StatusOK, users.PublicUsers())}func (s *Users) GetUser(c *gin.Context) {userId, err := strconv.ParseUint(c.Param("user_id"), 10, 64)if err != nil {c.JSON(http.StatusBadRequest, err.Error())return}user, err := s.us.GetUser(userId)if err != nil {c.JSON(http.StatusInternalServerError, err.Error())return}c.JSON(http.StatusOK, user.PublicUser())}
```

观察到返回用户时，我们仅返回一个公共用户（在实体中定义）。公共用户没有敏感的用户详细信息，例如电子邮件和密码。

 **授权处理**

login\_handler负责登录，注销和刷新令牌方法。在各自文件中定义的某些方法在此文件中被调用。最好在它们的文件路径之后在存储库中检出它们。

```
package interfacesimport ("fmt""food-app/application""food-app/domain/entity""food-app/infrastructure/auth""github.com/dgrijalva/jwt-go""github.com/gin-gonic/gin""net/http""os""strconv")type Authenticate struct {us application.UserAppInterfacerd auth.AuthInterfacetk auth.TokenInterface}//Authenticate constructorfunc NewAuthenticate(uApp application.UserAppInterface, rd auth.AuthInterface, tk auth.TokenInterface) *Authenticate {return &Authenticate{us: uApp,rd: rd,tk: tk,}}func (au *Authenticate) Login(c *gin.Context) {var user *entity.Uservar tokenErr = map[string]string{}if err := c.ShouldBindJSON(&user); err != nil {c.JSON(http.StatusUnprocessableEntity, "Invalid json provided")return}//validate request:validateUser := user.Validate("login")if len(validateUser) > 0 {c.JSON(http.StatusUnprocessableEntity, validateUser)return}u, userErr := au.us.GetUserByEmailAndPassword(user)if userErr != nil {c.JSON(http.StatusInternalServerError, userErr)return}ts, tErr := au.tk.CreateToken(u.ID)if tErr != nil {tokenErr["token_error"] = tErr.Error()c.JSON(http.StatusUnprocessableEntity, tErr.Error())return}saveErr := au.rd.CreateAuth(u.ID, ts)if saveErr != nil {c.JSON(http.StatusInternalServerError, saveErr.Error())return}userData := make(map[string]interface{})userData["access_token"] = ts.AccessTokenuserData["refresh_token"] = ts.RefreshTokenuserData["id"] = u.IDuserData["first_name"] = u.FirstNameuserData["last_name"] = u.LastNamec.JSON(http.StatusOK, userData)}func (au *Authenticate) Logout(c *gin.Context) {//check is the user is authenticated firstmetadata, err := au.tk.ExtractTokenMetadata(c.Request)if err != nil {c.JSON(http.StatusUnauthorized, "Unauthorized")return}//if the access token exist and it is still valid, then delete both the access token and the refresh tokendeleteErr := au.rd.DeleteTokens(metadata)if deleteErr != nil {c.JSON(http.StatusUnauthorized, deleteErr.Error())return}c.JSON(http.StatusOK, "Successfully logged out")}//Refresh is the function that uses the refresh_token to generate new pairs of refresh and access tokens.func (au *Authenticate) Refresh(c *gin.Context) {mapToken := map[string]string{}if err := c.ShouldBindJSON(&mapToken); err != nil {c.JSON(http.StatusUnprocessableEntity, err.Error())return}refreshToken := mapToken["refresh_token"]//verify the tokentoken, err := jwt.Parse(refreshToken, func(token *jwt.Token) (interface{}, error) {//Make sure that the token method conform to "SigningMethodHMAC"if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])}return []byte(os.Getenv("REFRESH_SECRET")), nil})//any error may be due to token expirationif err != nil {c.JSON(http.StatusUnauthorized, err.Error())return}//is token valid?if _, ok := token.Claims.(jwt.Claims); !ok && !token.Valid {c.JSON(http.StatusUnauthorized, err)return}//Since token is valid, get the uuid:claims, ok := token.Claims.(jwt.MapClaims)if ok && token.Valid {refreshUuid, ok := claims["refresh_uuid"].(string) //convert the interface to stringif !ok {c.JSON(http.StatusUnprocessableEntity, "Cannot get uuid")return}userId, err := strconv.ParseUint(fmt.Sprintf("%.f", claims["user_id"]), 10, 64)if err != nil {c.JSON(http.StatusUnprocessableEntity, "Error occurred")return}//Delete the previous Refresh TokendelErr := au.rd.DeleteRefresh(refreshUuid)if delErr != nil { //if any goes wrongc.JSON(http.StatusUnauthorized, "unauthorized")return}//Create new pairs of refresh and access tokensts, createErr := au.tk.CreateToken(userId)if createErr != nil {c.JSON(http.StatusForbidden, createErr.Error())return}//save the tokens metadata to redissaveErr := au.rd.CreateAuth(userId, ts)if saveErr != nil {c.JSON(http.StatusForbidden, saveErr.Error())return}tokens := map[string]string{"access_token":  ts.AccessToken,"refresh_token": ts.RefreshToken,}c.JSON(http.StatusCreated, tokens)} else {c.JSON(http.StatusUnauthorized, "refresh token expired")}}
```

6

**运行程序**

我们测试一下该应用。我们将连接路由，连接到数据库并启动应用程序。

在根目录中定义的main.go文件中完成。

```
package mainimport ("food-app/infrastructure/auth""food-app/infrastructure/persistence""food-app/interfaces""food-app/interfaces/fileupload""food-app/interfaces/middleware""github.com/gin-gonic/gin""github.com/joho/godotenv""log""os")func init() {//To load our environmental variables.if err := godotenv.Load(); err != nil {log.Println("no env gotten")}}func main() {dbdriver := os.Getenv("DB_DRIVER")host := os.Getenv("DB_HOST")password := os.Getenv("DB_PASSWORD")user := os.Getenv("DB_USER")dbname := os.Getenv("DB_NAME")port := os.Getenv("DB_PORT")//redis detailsredis_host := os.Getenv("REDIS_HOST")redis_port := os.Getenv("REDIS_PORT")redis_password := os.Getenv("REDIS_PASSWORD")services, err := persistence.NewRepositories(dbdriver, user, password, port, host, dbname)if err != nil {panic(err)}defer services.Close()services.Automigrate()redisService, err := auth.NewRedisDB(redis_host, redis_port, redis_password)if err != nil {log.Fatal(err)}tk := auth.NewToken()fd := fileupload.NewFileUpload()users := interfaces.NewUsers(services.User, redisService.Auth, tk)foods := interfaces.NewFood(services.Food, services.User, fd, redisService.Auth, tk)authenticate := interfaces.NewAuthenticate(services.User, redisService.Auth, tk)r := gin.Default()r.Use(middleware.CORSMiddleware()) //For CORS//user routesr.POST("/users", users.SaveUser)r.GET("/users", users.GetUsers)r.GET("/users/:user_id", users.GetUser)//post routesr.POST("/food", middleware.AuthMiddleware(), middleware.MaxSizeAllowed(8192000), foods.SaveFood)r.PUT("/food/:food_id", middleware.AuthMiddleware(), middleware.MaxSizeAllowed(8192000), foods.UpdateFood)r.GET("/food/:food_id", foods.GetFoodAndCreator)r.DELETE("/food/:food_id", middleware.AuthMiddleware(), foods.DeleteFood)r.GET("/food", foods.GetAllFood)//authentication routesr.POST("/login", authenticate.Login)r.POST("/logout", authenticate.Logout)r.POST("/refresh", authenticate.Refresh)//Starting the applicationapp_port := os.Getenv("PORT") //using heroku hostif app_port == "" {app_port = "8888" //localhost}log.Fatal(r.Run(":"+app_port))}
```

 其中的中间件也是定义在 interfaces 层。

```
package middlewareimport ("bytes""food-app/infrastructure/auth""github.com/gin-gonic/gin""io/ioutil""net/http")func AuthMiddleware() gin.HandlerFunc {return func(c *gin.Context) {err := auth.TokenValid(c.Request)if err != nil {c.JSON(http.StatusUnauthorized, gin.H{"status": http.StatusUnauthorized,"error":  err.Error(),})c.Abort()return}c.Next()}}func CORSMiddleware() gin.HandlerFunc {return func(c *gin.Context) {c.Writer.Header().Set("Access-Control-Allow-Origin", "*")c.Writer.Header().Set("Access-Control-Allow-Credentials", "true")c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization, accept, origin, Cache-Control, X-Requested-With")c.Writer.Header().Set("Access-Control-Allow-Methods", "POST, OPTIONS, GET, PUT, PATCH, DELETE")if c.Request.Method == "OPTIONS" {c.AbortWithStatus(204)return}c.Next()}}//Avoid a large file from loading into memory//If the file size is greater than 8MB dont allow it to even load into memory and waste our time.func MaxSizeAllowed(n int64) gin.HandlerFunc {return func(c *gin.Context) {c.Request.Body = http.MaxBytesReader(c.Writer, c.Request.Body, n)buff, errRead := c.GetRawData()if errRead != nil {//c.JSON(http.StatusRequestEntityTooLarge,"too large")c.JSON(http.StatusRequestEntityTooLarge, gin.H{"status":     http.StatusRequestEntityTooLarge,"upload_err": "too large: upload an image less than 8MB",})c.Abort()return}buf := bytes.NewBuffer(buff)c.Request.Body = ioutil.NopCloser(buf)}}
```

 我们现在可以使用以下命令运行该应用：

```
go run main.go
```

**总结**

希望通过构建该golang应用，帮助大家了解如何使用DDD。如果有什么疑问或意见，可以在下方留言。

360云计算

由360云平台团队打造的技术分享公众号，内容涉及数据库、大数据、微服务、容器、AIOps、IoT等众多技术领域，通过夯实的技术积累和丰富的一线实战经验，为你带来最有料的技术分享

![图片](https://mmbiz.qpic.cn/mmbiz_png/6MZqUI3KNlpqkSeOQoh6jDUAibq4lRENB0TJqNZwYPic6ZZcb87R98l9jSg5jA0ibJZEGDvNVtmazmyOH1ibbA677w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)