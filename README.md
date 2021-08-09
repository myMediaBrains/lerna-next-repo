# [Creating Next.js monorepo](https://medium.com/wesionary-team/creating-next-js-monorepo-d41ea78f4afb)

[Next.js 참조:](https://nextjs.org/docs/getting-started) 

| No   | 구분                                                  | 설명                  |
| ---- | ----------------------------------------------------- | --------------------- |
| 1    | $ npm i lerna -g                                      | lerna 설치            |
| 2    | $ mkdir lerna-next-repo && cd lerna-next-repo         | 프로젝트 폴더 생성    |
| 3    | $ lerna init                                          | monorepo 생성         |
| 4    | $ cd packages                                         |                       |
|      | $ lerna create shared                                 | shared 폴더 생성      |
|      | $ cd shared                                           | shared 폴더로 이동    |
|      | $ rm -rf \__tests__                                   | \__tests__ 폴더 제거  |
|      | $ rm -rf lib                                          | lib 폴더 제거         |
|      | $ mkdir src                                           | shared 폴더 생성      |
|      | $ cd src && touch index.ts                            | index.ts 생성         |
|      | $ cd .. && touch package.json                         | package.json 업데이트 |
|      | $ yarn add @types/react dotenv next-images typescript |                       |

packages/shared/package.json

```json
{
  "name": "shared",
  "version": "0.0.0",
  "description": "> TODO: description",
  "author": "myMediaBrains <my.media.brains@gmail.com>",
  "homepage": "",
  "license": "ISC",
  "main": "src/shared.js",
  "directories": {
    "src": "src"
  },
  "files": [
    "src"
  ],
  "scripts": {
    "test": "echo \"Error: run tests from root\" && exit 1"
  }
}
```

```json
{
  "name": "@monorepo/shared",
  "version": "0.0.0",
  "description": "> TODO: description",
  "author": "myMediaBrains <my.media.brains@gmail.com>",
  "homepage": "",
  "license": "ISC",
  "main": "src/index.ts",
  "dependencies": {
    "@types/react": "^17.0.16",
    "dotenv": "^10.0.0",
    "next-images": "^1.8.1",
    "typescript": "^4.3.5"
  }
}
```

| no   | 구분                                  | 설명           |
| ---- | ------------------------------------- | -------------- |
| 5    | $ cd packages && yarn create next-app | user 폴더 생성 |
| 6    | $ touch package.json                  | root 폴더      |

package.json

```json
{
  "name": "root",
  "private": true,
  "devDependencies": {
    "lerna": "^4.0.0"
  },
  "dependencies": {
    "next": "^11.0.1"
  }
}

```

```json
{
  "name": "root",
  "private": true,
  "devDependencies": {
    "lerna": "^4.0.0"
  },
  "scripts": {
    "bootstrap": "yarn install; lerna bootstrap;",
    "start": "lerna run start --parallel",
    "start:user": "node -r ./dotenv.config.js node_modules/.bin/lerna run --scope user --stream dev",
    "build:user": "node -r ./dotenv.config.js node_modules/.bin/lerna run --scope user --stream build",
    "run:build:user": "lerna run start --scope user"
  },
  "workspaces": [
    "packages/*"
  ]
}
```

우리가 workspaces 를 사용하고 있기 때문에, shared package 를 우리의 next 프로젝트인 user 에서 dependency 로 추가한다.

| no   | 구분               | 설명      |
| ---- | ------------------ | --------- |
| 7    | $ touch lerna.json | root 폴더 |

lerna.json

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "0.0.0"
}
```

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "0.0.0",
  "npmClient": "yarn",
  "useWorkspaces": true
}
```

| no   | 구분                 | 설명      |
| ---- | -------------------- | --------- |
| 8    | $ touch package.json | user 폴더 |

packages/user/package.json

```json
{
  "name": "user",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "11.0.1",
    "react": "17.0.2",
    "react-dom": "17.0.2"
  },
  "devDependencies": {
    "eslint": "7.32.0",
    "eslint-config-next": "11.0.1"
  }
}
```

```json
{
  "name": "user",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@monorepor/shared": "0.0.0",
    "next": "11.0.1",
    "react": "17.0.2",
    "react-dom": "17.0.2"
  },
  "devDependencies": {
    "eslint": "7.32.0",
    "eslint-config-next": "11.0.1"
  }
}

```

| no   | 구분                     | 설명      |
| ---- | ------------------------ | --------- |
| 9    | $ touch dotenv.config.js | root 폴더 |

dotenv.config.js

```tsx
const dotenv = require("dotenv");
const path = require("path");
if (process.env.NODE_ENV === "production") {
  dotenv.config({
    path: path.resolve(__dirname, `./.env.production`),
  });
} else {
  dotenv.config({
    path: path.resolve(__dirname, "./.env"),
  });
}
```

| no   | 구분                                             | 설명      |
| ---- | ------------------------------------------------ | --------- |
| 10   | $ lerna add next-compose-plugins \--scope=user   | user 폴더 |
|      | $ lerna add next-transpile-modules \--scope=user |           |
|      | $ lerna add next-images \--scope=user            |           |
| 11   | $ touch next.config.js                           | user 폴더 |

packages/user/next.config.js

```tsx
module.exports = {
  reactStrictMode: true,
}
```

```tsx
const withPlugins = require("next-compose-plugins");
const withTM = require("next-transpile-modules")(["@monorepo/shared"]);
const withImages = require("next-images");
module.exports = withPlugins([withTM(), withImages], {
  webpack: (config) => {
    // custom webpack config
    return config;
  },
  images: {},
});
```

| no   | 구분              | 설명    |
| ---- | ----------------- | ------- |
| 12   | $ yarn start:user | 앱 실행 |

![img](https://miro.medium.com/max/562/1*O9SDpwPaPy_LC6aZ-KYmlw.png)

| no   | 구분                     | 설명 |
| ---- | ------------------------ | ---- |
| 13   | $ lerna add typescript   | root |
|      | $ lerna add @types/react |      |
| 14   | $ touch tsconfig.json    | root |

tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": false,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve"
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"],
}
```

| no   | 구분                  | 설명      |
| ---- | --------------------- | --------- |
| 15   | $ touch tsconfig.json | user 폴더 |

packages/user/tsconfig.json

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "isolatedModules": true,
    "noEmit": true,
    "allowSyntheticDefaultImports": true
  }
}
```

| no   | 구분                 | 설명      |
| ---- | -------------------- | --------- |
| 16   | .js  .jsx > .ts .tsx | user 폴더 |

