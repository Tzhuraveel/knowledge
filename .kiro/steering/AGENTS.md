
## Database Migrations

Always create Prisma migrations with the Nx target, never by hand-crafting a migration folder or timestamp:

```
npx nx run @jsa/db-client-layer:migrate --name=<migration_name>
```

- This target runs `prisma migrate dev --create-only --name <name>` (cwd `libs/db-client-layer`). Because of `--create-only`, it generates the `migration.sql` file WITHOUT applying it — edit the generated SQL, then apply it.
- Apply locally with `prisma migrate dev`; in higher environments use the `deploy` target (`prisma migrate deploy`).
- Let Prisma own the folder name and timestamp — do not create them manually.
- Follow the repo git rule: decompose multi-step shell operations into explicit steps.
