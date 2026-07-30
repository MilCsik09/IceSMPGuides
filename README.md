# IceSMP

> *Két mondat rombolta le a régi világot. A harmadikat senki élő nem akarja hallani.*

Egy megrepedt Élet Fája, négy egymásnak feszülő hatalom és üresen álló trónok:
az **IceSMP** egy magyar nyelvű, történetvezérelt fantasy SMP, ahol a következő
korszakot a játékosok írják. Ez a repó a világot működtető plugin forrása.

Technikailag: Folia-alapú Minecraft 1.21.11 plugin Java 21-hez. Egy
fantasy SMP szerver játékmenet-rendszereit fogja össze: frakciók, kasztok,
specializációk, képességek, szakmák, gazdaság, küldetések, események,
területek, relikviák, társak és adminisztráció.

## Dokumentáció

Az embernek szánt, mérvadó dokumentáció öt kézikönyvből áll:

| Kézikönyv | Mire való? |
|---|---|
| [Minden funkció](docs/FEATURES.md) | Az aktív, részleges, letiltott és tervezett rendszerek teljes katalógusa |
| [Legújabb változások](docs/LATEST_CHANGES.md) | A futóként átadott JAR és az integrált release közötti eltérések, plugin-rollout |
| [Játékos kézikönyv](docs/PLAYER_GUIDE.md) | Kezdés, gameplay, parancsok és játékosoldali korlátok |
| [Builder kézikönyv](docs/BUILDER_GUIDE.md) | Világhelyszínek, crate-ek, NPC-k, régiók és builder-checklistek |
| [Admin kézikönyv](docs/ADMIN_GUIDE.md) | Admin/moderáció, parancsok, permissionök, GUI-k, config, recovery és playtest |

A teljes command-, route-, alias-, permission-, config-, message- és
komponensleltárt a **Repository Docs Inventory** GitHub Actions workflow
letölthető artifactja készíti. Ez gépi ellenőrzési réteg, nem hatodik
kézikönyv.

Kiegészítő belső források:

- [architektúra](docs/ARCHITECTURE.md);
- [resource pack modelljegyzék](docs/RESOURCE_PACK_CMD.md);
- [lore-kódex](docs/LORE.md) és [technikai lore-megfeleltetés](docs/LORE_REFERENCE.md);
- [fejlesztési roadmap és ötletbank](ROADMAP.md);
- [Discord- és képi teaser-csomag](docs/TEASER.md).

Az aktív dokumentáció minden fenti Markdown-oldala szó szerint tükröződik
az `IceSMPGuides` repóba, azonos könyvtárszerkezettel. Régi auditnaplók és
párhuzamos redirect-oldalak nem részei az aktív dokumentációs készletnek.
