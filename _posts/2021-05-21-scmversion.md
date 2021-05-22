---
title: Automatyczne wersjonowanie projektu w Gradle
author: Maciej Adamski
---

## Wstęp

### Wymagania
Do działania automatycznego wersjonowania potrzebujemy:

 - Projektu na zdalnym repozytorium GIT
 - Połączone konto ssh z tym repozytorium
 - gradle
 - axion-release-plugin

## Instalacja
###  Dodanie pluginu *axion-release-plugin*

By korzystać z *axion-release-plugin* musimy w sekcji *plugins* *gradla* dodać wpis informujący narzędzie, że będzie korzystało z naszej wtyczki do wersjonowania:
```
plugins {
    id 'pl.allegro.tech.build.axion-release' version '1.13.1'
}
```
### Zmiana wersji projektu na dynamicznie ustawianą przez nasz plugin

Wtyczka wystawia zmienną `scmVersion.version`, który zawiera aktualny numer wersji naszego projektu. By była ona wykorzystywana podczas budowania, musimy poinformować narzędzie *gradle*, że to w niej powinna szukać odpowiedniego numer wersji i z niego skorzystać. By to osiągnąć należy zmienić statyczną wersje  `project.version = X.Y.Z`, na zmienną związaną z *pluginem* `project.version = scmVersion.version`. W tym momencie podczas budowania projektu używany będzie numer wersji zapisany w tym parametrze *scmVersion*.

## Konfiguracja
Przedstawię jedynie najważniejsze opcje konfiguracyjne. Pozostałe można znaleźć na oficjalnej stronie *axion-release-plugin*.
Ustawienia pluginu możemy zmieniać w objekcie *scmVersion*, który należy najpierw dodać do pliku *build.gradle*.
```
scmVersion {
	apply plugin: "pl.allegro.tech.build.axion-release"
	...
}
```

### Prefixy wersji
*Axion-release-plugin* pozwala nam na ustawienie prefixów tagów wersji globalnie, jak i dla poszczególnych branchy osobno. W objekcie *scmVersion* należy dodać objekt *tag*, a w nim odpowiednie prefixy.
Dodawanie prefixu globalnego wymaga tylko parametru *prefix* w *tag*:
```
tag {
	prefix: 'pw.pik.blog-'
}
```
Tak przygotowany tag przy wydawaniu projektu będzie przekształcony w `pw.pik.blog-X.Y.Z`, gdzie x, y i z to odpowiednie numery wersji.
By dodać prefixy dla konkretnych branchy możemy skorzystać z tablicy *branchPrefix*, np.:
```
tag {
	prefix: 'pw.pik.blog-'
	branchPrefix: [
		'develop.*': 'pw.pik.blog.develop-'
	]
}
```
Takie ustawienie sprawi, że globalnie będziemy korzystać z nazwy wersji `pw.pik.blog-X.Y.Z`, natomiast dla branchy `develop*` z wersji `pw.pik.blog.develop-X.Y.Z`. Możemy tak dodać wszystkie interesujące nas branche i odpowiednie dla nich nazwy wydań. Jeśli jakiś branch nie zostanie dopasowany to skorzysta on po prostu z globalnego prefixu.

### Repozytorium

W konfiguracji pluginu możemy dodać objekt *repository*, który służy do ustawień parametrów związanych z repozytorium kodu, na którym znajduje się projekt.
```
repository {
	...
}
```
#### Katalog główny
Domyślnie plugin *axion* uznaje, że plik *build.gradle* znajduje się w katalogu głównym naszego repozytorium. Jeśli w rzeczywistości jest inaczej i nasz plik budowania znajduje się np. w folderze *backend* musimy poinformować narzędzie gdzie znajduje się katalog *root*.
```
repository {
	directory = project.rootProject.file('<PATH>')
}
``` 
W przytoczonym przykładzie *build.gradle* znajduje się w `<root>/backend`, więc `<PATH>` musimy ustawić na cofnięcie się jeden poziom w górę: `project.rootProject.file('..')`

#### Problemy "detached head"
W niektórych przypadkach może wystąpić kłopot przy stanie *detached head*, gdy próbujemy wystawić wersję z lokalnego repozytorium na zdalne. By rozwiązać ten problem, musimy poinformować plugin by *pushował* jedynie tagi. Osiągniemy to dodając pole *pushTagsOnly* do objektu *repository*
```
repository {
	pushTagsOnly: true
}
```

## Działanie

### Użyta konfiguracja
W prezentacji działania narzędzie *scmVersion* korzystam z następującej konfiguracji:
```
scmVersion {	
    apply plugin: "pl.allegro.tech.build.axion-release"
    repository {
        directory = project.rootProject.file('..')
		pushTagsOnly = true
    }
	tag {
        prefix = 'pw.pik.blog-'
    }
}
```

 - *directory* - ustawione na folder wyżej w hierarchi, ponieważ *build.gradle* znajduje się w `<root>/backend`
 - *tag->prefix* - wersje będą oznaczne *pw.pik.blog-X.Y.Z*
- narzędzie będzie wypchać jedynie tagi

### Prezentacja działania

Aktualnie w projekcie, na którym będę prezentować zmiany znajdują się następujące tagi (oznaczjące wydane wersje):
```
~git tag
pw.pik.blog-0.1.0
pw.pik.blog-0.1.1
```

&nbsp;
Sprawdźmy w narzędziu do budowania aktualną wersję projektu komendą `./gradlew currentVersion`.
```
~./gradlew currentVersion

> Task :currentVersion
Project version: 0.1.1
```
&nbsp;
Jak widać aktualna wersja projektu `0.1.1` zgadza się z tagami. Najnowszy tag bowiem miał postać: `pw.pik.blog-0.1.1` i od tamtego czasu nie zostały zcommitowane żadne zmiany w projekcie.

Wprowadźmy dowolne zmiany w projekcie , a następnie je zcommitujmy:
```
git commit -m "Add some changes - blog new ver"
[main 61f106d] Add some changes - blog new ver
 1 file changed, 2 deletions(-)
```

&nbsp;
Sprawdźmy czy wersja naszego projektu się zmieniła:
```
~./gradlew currentVersion

> Task :currentVersion
Project version: 0.1.2-SNAPSHOT
```
&nbsp;
Wersja naszego projektu teraz to `0.1.2-SNAPSHOT`. *SNAPSHOT* w wersji oznacza, że od czasu ostatniego wydania zostały wprowadzone zmiany robocze (ale nie zostały wydane). 
Jeśli teraz byśmy spróbowali wydać wersję poleceniem `./gradlew release` to skończyłoby się to niepowodzeniem ponieważ zmiany nie został wypchnięte na repozytorium zdalne (można ustawić specjalną opcję, by można było wydawać wersję bez wypychania - w tym wypadku nie skorzystaliśmy z tej opcji).

Wypchnijmy więc najnowszy commit na repozytorium zdalne:
```
git push origin
[...]
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com:madamskip1/PIK_BLOG_SCMVERSION.git
   90b4b65..61f106d  main -> main
   ```
   
   &nbsp;
Sprawdzając poleceniem `./gradlew currentVersion` możemy sprawdzić, że wypchnięcie danych na repozytorium nie miało jeszcze wpłwu na wersję:
```
~./gradlew currentVersion

> Task :currentVersion
Project version: 0.1.2-SNAPSHOT
```

&nbsp;
W tej chwili lokalny commit jest tym samym co commit zdalny - możemy więc wydać wersję poleceniem `./gradlew release`
```
~./gradlew release

> Task :verifyRelease
Looking for uncommitted changes..
Checking if branch is ahead of remote..
Checking for snapshot versions..

> Task :release
Creating tag: pw.pik.blog-0.1.2
Pushing all to remote: origin

BUILD SUCCESSFUL in 3s
```
&nbsp;
Jak widać udało się utworzyć nowy tag i wydać wersję - nazywa się ona teraz `pw.pik.blog-0.1.2`.
Upewnijmy się czy w istocie tak się stało:
```
~git tag
pw.pik.blog-0.1.0
pw.pik.blog-0.1.1
pw.pik.blog-0.1.2
```
```
~./gradlew currentVersion
> Task :currentVersion
Project version: 0.1.2
```
Jak widzimy zarówno tagi repozytorium jak i wersja projektu w *gradle* pokazują ten sam numer - *0.1.2*. Oznacza to, że udało się nam wydać kolejną wersję projektu.


## Podsumowanie
Przedstawiona narzędzie jakim jest *scmVersion* w automatyczny sposób pomaga nam zwiększać numery wersji projektów i aplikacji, bez konieczności ręcznego zmieniania każdorazowo numeru w odpowiednich plikach.
Jest to bardzo istotne w filozofii CI/CD, gdzie kolejne wersje wydajemy dosyć często w sposób automatyczny - możemy wtedy ustawić w programie do zarządzania ciągłą integracją i ciągłym dostarczaniem, by przy aktualizacji kodu i deployowaniu aplikacji sam automatycznie zmieniał wersję uruchamiając komendę *./gradlew release*, dzięki czemu programiści i osoby odpowiedzialne za projekt mogą skupić się na budowaniu aplikacji i dostarczaniu treści.
