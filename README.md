# GORM Sqlite Driver

![CI](https://github.com/go-gorm/sqlite/workflows/CI/badge.svg)

## USAGE

### 主要用来适配gorm和go-sqlcipher的

```go
package sqlcipher

import (
	"encoding/json"
	"fmt"
	"testing"

	"gorm.io/gorm"
)

type Code struct {
	Id     int64
	Code   *string
	Status *string
}

func (Code) TableName() string {
	return "code"
}

func (c Code) String() string {
	bts, _ := json.Marshal(c)
	return string(bts)
}

func TestNormal(t *testing.T) {

	db, _ := gorm.Open(Open("normal.db"), nil)

	db.Exec(`CREATE TABLE IF NOT EXISTS "code" (
	  "id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
	  "code" TEXT(64) NOT NULL DEFAULT '',
	  "status" integer(11) NOT NULL DEFAULT 1
	);`)

	db.Exec(`INSERT INTO code(code,status) VALUES('123', 1)`)

	codes := make([]*Code, 0)
	err := db.Find(&codes).Error
	if err != nil {
		panic(err)
	}

	fmt.Println(codes)
}

func TestEncrypt(t *testing.T) {

	/** SQLCipher 3 默认值
	PRAGMA cipher_page_size = 1024;
	PRAGMA kdf_iter = 4000;
	PRAGMA cipher_kdf_algorithm = PBKDF2_HMAC_SHA1;
	PRAGMA cipher_hmac_algorithm = HMAC_SHA1;
	*/

	/**
	SQLCipher 4 默认值,好多软件打不开db文件
	PRAGMA cipher_page_size = 4096;
	PRAGMA kdf_iter = 256000;
	PRAGMA cipher_kdf_algorithm = PBKDF2_HMAC_SHA512;
	PRAGMA cipher_hmac_algorithm = HMAC_SHA512;
	*/

	// 1、参数的方式(修改版本)
	// github.com/dm764/go-sqlcipher/v4 v4.4.3
	// 下面这种加密方式，只适合v4.4.3版本
	dbname := "encrypt.db?" +
		"_pragma_key=123456" +
		"&_pragma_cipher_page_size=1024" +
		"&_pragma_kdf_iter=64000" +
		"&_pragma_cipher_kdf_algorithm=PBKDF2_HMAC_SHA1" +
		"&_pragma_cipher_hmac_algorithm=HMAC_SHA1" +
		"&_pragma_cipher_use_hmac=OFF"
	db, _ := gorm.Open(Open(dbname), nil)

	err := db.Exec(`CREATE TABLE IF NOT EXISTS "code" (
	  "id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
	  "code" TEXT(64) NOT NULL DEFAULT '',
	  "status" integer(11) NOT NULL DEFAULT 1
	);`).Error

	if err != nil {
		t.Fatal(err)
	}

	err = db.Exec(`INSERT INTO code(code,status) VALUES('123', 1)`).Error
	if err != nil {
		t.Fatal(err)
	}

	codes := make([]*Code, 0)
	err = db.Find(&codes).Error
	if err != nil {
		t.Fatal(err)
	}

	fmt.Println(codes)

}


```


