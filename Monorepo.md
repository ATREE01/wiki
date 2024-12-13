# PNPM Workspace (Not Yet finished.)
1. **Pnpm**
``` bash
$ pnpm init
$ vim pnpm-workspace.yaml
```

```yaml
# pnpm-workspace.yaml
packages: 
    apps/*
```

2. **Lerna**
```bash
$ npx lerna init
(set npmClient as pnpm )
```

3. **CREATE APPS**
```bash
$ cd apps
$ npx create-next-app@latest
$ npm i -g @nestjs/cli
$ nest new {project-name}
```

4. **Install DevDeps**
```bash
# husky
$ pnpm add husky -D
$ npx husky init

# eslint
$ pnpm add eslint
$ touch .eslintrc.js

# lint-staged
$ pnpm add lint-staged
```


