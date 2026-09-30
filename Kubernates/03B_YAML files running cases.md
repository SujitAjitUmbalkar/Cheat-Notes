# Running YAML files multiple times 


| Situation                             | Pod                                                        | ReplicaSet                                             | Deployment                                      |
| ------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------- |
| First `apply`                         | Creates Pod                                                | Creates RS + Pods                                      | Creates Deployment + RS + Pods                  |
| Identical `apply` again               | No new Pod                                                 | No new Pods                                            | No new Pods                                     |
| Increase `replicas`                   | ❌                                                          | ✅ Creates more                                         | ✅ Creates more                                  |
| Decrease `replicas`                   | ❌                                                          | ✅ Removes Pods                                         | ✅ Removes Pods                                  |
| Pod dies                              | ❌ No replacement                                           | ✅ Replacement                                          | ✅ Replacement                                   |
| Change **container name**             | ⚠️ Existing Pod usually must be recreated                  | ⚠️ Template updates, existing Pods stay unchanged      | ✅ New ReplicaSet + rollout                      |
| Change **container image/version**    | ⚠️ Existing Pod doesn't automatically become a new version | ⚠️ Template updates, existing Pods stay on old version | ✅ New ReplicaSet + rollout                      |
| Change **container port**             | ⚠️ Existing Pod generally must be recreated                | ⚠️ Template updates, existing Pods stay unchanged      | ✅ New ReplicaSet + rollout                      |
| Change **environment variable**       | ⚠️ Existing Pod generally must be recreated                | ⚠️ Template updates, existing Pods stay unchanged      | ✅ New ReplicaSet + rollout                      |
| Change **resource limits/requests**   | ⚠️ Usually requires Pod recreation                         | ⚠️ Template updates, existing Pods stay unchanged      | ✅ New ReplicaSet + rollout                      |
| Change Pod **labels/template**        | ⚠️ Usually recreate                                        | ⚠️ Template changes, existing Pods stay unchanged      | ✅ New ReplicaSet + rollout                      |
| Change `replicas: 3 → 5`              | ❌ Doesn't manage replicas                                  | ✅ 2 Pods added                                         | ✅ 2 Pods added                                  |
| Change `replicas: 5 → 2`              | ❌                                                          | ✅ 3 Pods removed                                       | ✅ 3 Pods removed                                |
| Delete one Pod manually               | ❌ Stays deleted                                            | ✅ ReplicaSet creates replacement                       | ✅ Deployment/RS creates replacement             |
| Delete all Pods                       | ❌ No Pods                                                  | ✅ Creates desired number                               | ✅ Creates desired number                        |
| Delete ReplicaSet                     | ❌ Not applicable                                           | ❌ RS + its Pods disappear                              | ⚠️ Deployment normally recreates its managed RS |
| Change **Deployment image version**   | N/A                                                        | N/A                                                    | ✅ Creates new ReplicaSet and rolls out          |
| Rollback previous application version | ❌                                                          | ❌ No built-in rollout history                          | ✅ `kubectl rollout undo`                        |
| Rolling update                        | ❌                                                          | ❌                                                      | ✅                                               |
| Maintain desired Pod count            | ❌                                                          | ✅                                                      | ✅                                               |
| Manage ReplicaSets                    | ❌                                                          | ❌                                                      | ✅                                               |
