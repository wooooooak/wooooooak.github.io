---
layout: post
title: Cannot add foreign key constraint
category: error
tags: [sequelize, database]
comments: true
---

## 어떤 에러인가?

Node.js와 sequelize로 웹 서버, 데이터베이스를 구축했다. 실제 프로덕션 환경에서 사용할 데이터베이스는 aws RDS인데 개발중에 사용하면 괜히 돈이 나갈까봐 local환경의 데이터베이스에서 작업해왔다. local에서는 전혀 문제없이 원하는 모든 테이블이 정상적으로 생성되었는데, aws RDS에서 실행해보니

```SQL
CREATE TABLE IF NOT EXISTS `folders`
    (`id` CHAR(36) BINARY , `folderName` VARCHAR(255),
     `folderCoverImage` VARCHAR(255), `createdAt` DATETIME NOT NULL,
     `updatedAt` DATETIME NOT NULL, `fk_user_id` CHAR(36) BINARY,
     `fk_category_id` CHAR(36) BINARY,
    PRIMARY KEY (`id`),
    FOREIGN KEY (`fk_user_id`) REFERENCES `users` (`id`)
        ON DELETE CASCADE ON UPDATE RESTRICT,
    FOREIGN KEY (`fk_category_id`) REFERENCES `categories` (`id`)
        ON DELETE SET NULL ON UPDATE CASCADE) ENGINE=InnoDB;

Unhandled rejection SequelizeDatabaseError: Cannot add foreign key constraint
```

에러가 뜨는 것이다. 당황스럽기도 했고 신기하기도 했다. 분명히 동일한 코드에 동일한 DB(MYSQL)스펙인데.. 그저 host의 주소만 바뀐것 같은데 어떻게 이런일이 발생한 걸까?

## 상황

sequelize가 처리해주는 테이블 생성 statement는 아래와 같았다.

```SQL
Executing (default): CREATE TABLE IF NOT EXISTS `users` (`id` CHAR(36) BINARY , `username` VARCHAR(255), `email` VARCHAR(255) UNIQUE, `socialProvider` VARCHAR(255), `profileImg` TEXT, `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Executing (default): CREATE TABLE IF NOT EXISTS `posts` (`id` CHAR(36) BINARY , `postTitle` VARCHAR(255), `fk_user_id` CHAR(36) BINARY, `subTitle` TEXT, `editorState` TEXT, `bookCoverImg` VARCHAR(255), `like` INTEGER DEFAULT 0, `starRating` INTEGER DEFAULT 0, `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`), FOREIGN KEY (`fk_user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE ON UPDATE RESTRICT) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Executing (default): CREATE TABLE IF NOT EXISTS `categories` (`id` CHAR(36) BINARY , `name` VARCHAR(255) UNIQUE, `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Executing (default): CREATE TABLE IF NOT EXISTS `tags` (`id` CHAR(36) BINARY , `name` VARCHAR(255) UNIQUE, `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Executing (default): CREATE TABLE IF NOT EXISTS `folders` (`id` CHAR(36) BINARY , `folderName` VARCHAR(255), `folderCoverImage` VARCHAR(255), `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, `fk_user_id` CHAR(36) BINARY, `fk_category_id` CHAR(36) BINARY, PRIMARY KEY (`id`), FOREIGN KEY (`fk_user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE ON UPDATE RESTRICT, FOREIGN KEY (`fk_category_id`) REFERENCES `categories` (`id`) ON DELETE SET NULL ON UPDATE CASCADE) ENGINE=InnoDB;
Unhandled rejection SequelizeDatabaseError: Cannot add foreign key constraint
```

구글링을 꽤 오랫동안 해보았는데, 대부분의 원인은 테이블 생성의 순서인 경우가 많았다. 나의 코드로 예를 들자면, folders테이블은 users와 categories테이블을 참조해야하는데, 만약 folders 테이블이 먼저 생성된다면 folders의 foreign key를 설정하는 부분에서 user테이블 또는 categories테이블을 찾을 수 없어서 이런 에러가 나는 것이다.

그러나 아쉽게도 나의 경우는 아니었다. 순서는 정확했다. 코드를 봐도 알고 local환경에서는 잘 돌아가는 것을 보면 알 수 있다.

## 원인

결국 성공할 때와 실패할 때의 차이점은 database환경일 뿐이라는 결론을 내렸다. 열심히 stackoverflow에 질문하고 구글링한 결과 database의 설정이 다른게 맞았다. 참조하고 참조 당하는 두 테이블 간에 character set이 달랐던 것이다. 나는 바보같이, 다른 모델은 전부 utf-8로 설정해 놓고선 folder 모델을 정의할 때 character set을 utf-8로 지정하지 않았었다. 그래서 오류가 난 것이다.

#### 로컬에서는 왜 작동을까?

내가 설정한 건지 잘 기억나진 않지만, local의 데이터베이스 테이블들은 생성될 때 기본값으로 character set이 utf-8로 설정되게 되어있었다. 반면 새로 생성한 AWS RDS는 기본값이 latin 1이었다...

## 해결

우선 이를 해결하기 위해 간단히 folder모델에 charset을 정의해주었다.

```javascript
const Folder = sequelize.define(
  "folder",
  {
    id: {
      type: Sequelize.UUID,
      defaultValue: Sequelize.UUIDV1,
      primaryKey: true
    },
    folderName: {
      type: Sequelize.STRING
    },
    folderCoverImage: {
      type: Sequelize.STRING
    }
  },
  {
    timestamps: true,
    charset: "utf8"
  }
);
```

또는 이렇게 하지 않아도 aws RDS역시 기본값을 UTF-8로 주는 방법이 있다. 여기선 기록하지 않지만 찾아보면 금방이다.

참고로 현재 데이터베이스의 기본 CHARSET값을 알고싶다면 아래와 같이 명령문을 날리면 된다.

```
SHOW CREATE DATABASE mydb;
```

## 참고자료

- [Dealing with MySQL Error Code 1215: “Cannot add foreign key constraint”](https://www.percona.com/blog/2017/04/06/dealing-mysql-error-code-1215-cannot-add-foreign-key-constraint/)

- [Dealing with MySQL Error Code 1215: “Cannot add foreign key constraint” 번역](https://devcken.io/mysql-oryu-kodeu-1215-darugi/)

- [MySQL database 와 table 의 character set 확인하는 법](https://www.lesstif.com/pages/viewpage.action?pageId=17105743)
