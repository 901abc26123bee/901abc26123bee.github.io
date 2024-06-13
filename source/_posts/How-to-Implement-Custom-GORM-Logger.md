---
title: How to Implement Custom GORM Logger
date: 2024-06-13 22:32:35
categories:
tags:
- Database
- Golang
- GORM
---
# How to Implement Custom GORM Logger

GORM is a powerful ORM library for Go, widely used for database interactions. While GORM provides default logging capabilities, there are scenarios where you might need more customized logging solutions to suit your application's requirements.

1. Implement the interface
    ```go
    type Interface interface {
      LogMode(LogLevel) Interface
      Info(context.Context, string, ...interface{})
      Warn(context.Context, string, ...interface{})
      Error(context.Context, string, ...interface{})
      Trace(ctx context.Context, begin time.Time, fc func() (sql string, rowsAffected int64), err error)
    }
    ```
    implementation
    ```go
    package orm

    import (
      "context"
      "strings"
      "time"

      log "github.com/sirupsen/logrus"
      "gorm.io/gorm/logger"
      gorm_util "gorm.io/gorm/utils"
    )

    const UserUIDKey = "user_uid"
    const slowThreshold = 200 * time.Millisecond

    // CustomGormLogger defines custom gorm logger
    type CustomGormLogger struct { 
      logger logger.Interface
    }

    // NewCustomGormLogger new a gorm custom logger instance
    func NewCustomGormLogger() *CustomGormLogger {  
      return &CustomGormLogger{  
        logger: logger.Default,
      }  
    }

    // LogMode log mode
    func (l *CustomGormLogger) LogMode(lev logger.LogLevel) logger.Interface {  
      return l.logger.LogMode(lev)
    }  

    // Info print info
    func (l *CustomGormLogger) Info(ctx context.Context, msg string, data ...interface{}) {  
      l.logger.Info(ctx, msg, data)
    }

    // Warn print warn messages
    func (l *CustomGormLogger) Warn(ctx context.Context, msg string, data ...interface{}) {  
      l.logger.Warn(ctx, msg, data)
    }

    // Error print error messages
    func (l *CustomGormLogger) Error(ctx context.Context, msg string, data ...interface{}) {  
      l.logger.Error(ctx, msg, data) 
    }

    // Trace print custom sql message with uid
    func (l *CustomGormLogger) Trace(ctx context.Context, begin time.Time, fc func() (sql string, rowsAffected int64), err error) {
      userUid := ctx.Value(UserUIDKey)
      elapsed := float64(time.Since(begin))/1e6

      sql, rows := fc()
      formatSql := strings.ReplaceAll(sql, "\"", "")

      log.Infof("[file]=%s", gorm_util.FileWithLineNum())
      log.Infof("[%.3fms][rows %d][user_uid %s] %s", elapsed, rows, userUid, formatSql)

      if err != nil {  
        log.Warnf("[%.3fms] SQL ERROR, %v", elapsed, err)
      }  
      if elapsed > float64(slowThreshold.Nanoseconds()) {  
        log.Infof("[%.3fms] SLOW SQL", elapsed)  
      }
    }
    ```

2. Enable debug mode and replace with custom logger

    ```go
    // NewServer creates a server implementation for student server.
    func NewServer(ctx context.Context, config Config) (studentplanpb.StudentPlanServiceServer, error) {
      connectionString := fmt.Sprintf(db.CONNECTION_FORMAT_STRING, config.SQLHost, config.DBUser, config.DBPassword, config.DBPort, config.SQLDB)
      db, err := db.InitDB(connectionString)
      if err != nil {
        return nil, fmt.Errorf("failed to connect to mysqlDB: %v", err)
      }
      if config.DBDebug {
        db = db.Debug()
        defaultLogger := orm.CustomGormLogger()
        defaultLogger.LogMode(logger.Info)
        db.Logger = defaultLogger
      }

      s := &impl{
        Config:     config,
        db:         db,
        pool:       pool,
        stroageCli: stroageCli,
        stdriver:   stroageServer,
      }

      return s, nil
    }
    ```

3. Usage
    It is more better to implement ParseAuthorization as middleware and set UserUIDKey for custom GORM logging within the middleware to avoid code duplication.

    ```go
    func (s *impl) queryUserss(ctx context.Context, req *userpb.QueryUsersRequest) (*userpb.QueryUsersResponse, error) {
      // check user role
      uid, role := jwtext.ParseAuthorization(ctx, s.JWT)
      if role == userpb.Role_UNKNOWN {
        return nil, errors.GrpcError(codes.Unauthenticated, "user not logged in")
      }

      // set UserUIDKey for custom gorm logging
      ctx = context.WithValue(ctx, orm.UserUIDKey, uid)
    ```

4. Result

    ```go
    time="2024-05-16T14:11:23+08:00" level=info msg="starting student server at port :9090"
    time="2024-05-16T14:11:25+08:00" level=info msg="[file]=xxx/backend/v3/pkg/db/utils.go:264"
    time="2024-05-16T14:11:25+08:00" level=info msg="[16.995ms][rows 1][user_uid 435e7fa8-b444-11ee-96d1-3e9675b204c9] SELECT * FROM users WHERE id='435e7fa8-b444-11ee-96d1-3e9675b204c9' and deleted=0 ORDER BY users.id OFFSET 0 ROW FETCH NEXT 1 ROWS ONLY"
    ```
