# Setup: Dynatrace → GitHub Actions

## Paso 1: Revocar token expuesto
1. Ve a https://github.com/settings/tokens
2. Revoca el token `ghp_gdHoaSSO...`
3. Crea uno nuevo con scope `repo` (para workflow_dispatch)

## Paso 2: Configurar Credential en Dynatrace
1. Ve a **Settings → Credential Vault → Add credential**
2. Configura:
   - **Credential type**: Token
   - **Credential name**: `TOKEN_POC_RESTART_PODS`
   - **Credential scope**: ✅ Extension authentication
   - **Token value**: Tu nuevo token de GitHub

## Paso 3: Push del GitHub Action
```bash
cd test-wd-dyna
git add .github/workflows/dynatrace-test.yml
git commit -m "Add Dynatrace test workflow"
git push origin main
```

## Paso 4: Crear Workflow en Dynatrace
1. Ve a **Workflows → + Workflow**
2. Crea un workflow con trigger manual (para test)
3. Agrega task "HTTP Request" con:
   - **Method**: POST
   - **URL**: `https://api.github.com/repos/Icarouscja/test-wd-dynatrace/actions/workflows/dynatrace-test.yml/dispatches`
   - **Headers**:
     ```
     Accept: application/vnd.github+json
     Authorization: Bearer {{ .credential.TOKEN_POC_RESTART_PODS }}
     X-GitHub-Api-Version: 2022-11-28
     ```
   - **Body**:
     ```json
     {
       "ref": "main",
       "inputs": {
         "event_type": "manual_test",
         "pod_name": "test-pod",
         "namespace": "default",
         "problem_id": "TEST-001"
       }
     }
     ```

## Paso 5: Test
1. Ejecuta el workflow manualmente en Dynatrace
2. Verifica en GitHub → Actions que se disparó el workflow
3. Revisa los logs del job

## Siguiente: Trigger por eventos de Kubernetes
Una vez funcione el test, cambiaremos el trigger a:
- **Event trigger**: `events.kubernetes`
- Filtrar por: `event.type == "POD_RESTART"` o similar
