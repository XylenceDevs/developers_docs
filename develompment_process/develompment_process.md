# Guía de Buenas Prácticas de Ramas y Ambientes — Startup

> **Objetivo:** Que todos en el equipo trabajemos con el mismo flujo de ramas y ambientes, asegurando estabilidad en producción y rapidez en desarrollo.

---

## 1) Ramas principales

Usaremos **3 ramas permanentes**:

- **`main`** → código en **producción**, estable, probado.  
- **`test`** → ambiente de **staging/QA**, para probar antes de pasar a producción.  
- **`dev`** → ambiente de **desarrollo compartido**, donde se integran features antes de pasarlas a `test`.

**Diagrama:**
```
(feature branches) → dev → test → main
```

---

## 2) Flujo de trabajo diario

1. Crear una rama desde `dev`:
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b feat/login-con-google
   ```

2. Hacer commits pequeños y descriptivos:
   ```bash
   git commit -m "feat(auth): agrega login con Google"
   ```

3. Subir la rama y abrir un Pull Request hacia `dev`:
   ```bash
   git push origin feat/login-con-google
   ```

4. Revisar el PR (al menos 1 compañero aprueba).  
5. Merge a `dev` → despliegue automático al ambiente **dev**.  

---

## 3) Pasar a `test` (Staging/QA)

Cuando un conjunto de features esté listo para validar:

```bash
git checkout test
git pull origin test
git merge dev
git push origin test
```

- Esto dispara el **deploy en el ambiente test**.  
- QA revisa, se hacen pruebas con usuarios internos.  
- Si hay errores → corregir en `dev` y repetir.  

---

## 4) Pasar a `main` (Producción)

Cuando QA valida en `test`:

```bash
git checkout main
git pull origin main
git merge test
git push origin main
```

- Esto dispara el **deploy en producción**.  
- Crear un **tag de versión**:
  ```bash
  git tag -a v1.2.0 -m "Release v1.2.0"
  git push origin v1.2.0
  ```

---

## 5) Naming y convenciones

- **Ramas de feature:**  
  `feat/<nombre>` → `feat/api-reportes`  
- **Ramas de bugfix:**  
  `fix/<nombre>` → `fix/ui-boton-guardar`  
- **Ramas de hotfix (para prod):**  
  `hotfix/<nombre>` → `hotfix/fix-login`  

Commits siguen convención:
```
feat: agrega login con Google
fix: corrige bug en validación de email
chore: actualiza dependencias
```

---

## 6) CI/CD y ambientes

- **`dev`** → deploy automático al ambiente de desarrollo (pueden fallar cosas).  
- **`test`** → deploy automático al ambiente staging (todo debería funcionar, QA lo valida).  
- **`main`** → deploy manual/aprobado a producción.  

---

## 7) Hotfix en producción

1. Crear desde `main`:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b hotfix/fix-login
   ```
2. Corregir bug → PR → merge a `main`.  
3. Promocionar también a `test` y `dev` para que no se pierda el fix:
   ```bash
   git checkout dev
   git merge main
   git push origin dev

   git checkout test
   git merge main
   git push origin test
   ```

---

## 8) Resumen visual del flujo

```
feat/* → dev → test → main
             ↑       ↑
           QA ok   Deploy prod + tag
```

---

### 📌 Conclusión

- Todos trabajan en ramas cortas desde `dev`.  
- QA valida en `test`.  
- Producción solo se toca desde `main`.  
- Los nombres de ramas y commits son consistentes.  
- Siempre hay rollback posible gracias a los tags en `main`.  

Con este flujo, cualquier dev nuevo puede integrarse rápido y entender cómo trabajamos. 🚀
