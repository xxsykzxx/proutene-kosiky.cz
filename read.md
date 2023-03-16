# Instalace ABRA BI

Tento návod popisuje některé nezbytné kroky pro instalaci ABRA BI na systému Windows. Pro instalaci na OS Linux je třeba znalostí konkrétní distribuce os Linux a tyto kroky popisované pro Windows provézt správně, pro konkrétní distribuci Linux (Kroky se mohou lišit v různých distribucích)

## JAVA

Zajistit, aby byla nainstalovaná JAVA. Pro verzi ABRA BI 22 a vyšší je nutná **JAVA verze 11**. Aktuálně doporučovaná je JAVA 11 Open  JDK - vždy  64bit verze!! Zde jsou odkazy na místa:  
  

[Java  Adoptium  JDK 11](https://adoptium.net/temurin/releases/?version=11)

## Aplikační server  Apache  Tomcat

### Instalace
Pro provoz aplikace ABRA BI je možné použít téměř jakýkoliv JAVA aplikační server (Tomcat, JBoss, GlassFish, WebSphere, ...). Zde popsaný postup je pro Apache Tomcat v. 9 ke stažení [zde](https://tomcat.apache.org/download-90.cgi) . Vždy se doporučuje používat nejaktuálnější verzi Tomcat 9.

**Pokud instalujete Apache  Tomcat pomocí service  installeru v prostředí WIN (vždy 64bit) , zkontrolujte si, zda instalátor zvolil správnou verzi Java, pokud ne, opravte v instalačním dialogu cestu na správnou.**

### Nastavení 
Pro nastavení Apache Tomcat spusťte konfigurační program **Tomcat9w.exe** Vše podstatné se nachází na záložce Java.
 * Java Virtual Machine - zde je vyplněna cesta k JAVĚ, kterou bude Tomcat používat.
* Java Options - sem doplníme některé konfigurační parametry uvedené níže.
	-XX:CompressedClassSpaceSize=192m
	-XX:MaxMetaspaceSize=384m
    -Dfile.encoding=UTF8
    
    ***Nepovinné parametry***
    
	-Dabra.abrabi.thread.count=x  
	(kde ideálně x = (počet CPU *5) - nastavuje počet souběžně počítaných komponent, při vytváření stránky. Pokud není nastaveno je použito (počet CPU *4)

	Následují parametry ovlivňují využití paměti. Parametry Shenandoah GC (VIZ NÍŽE)  řídí práci s garbage  collection v Java a rychlost uvolňování operační paměti v situacích, kdy jí Java už dále nevyužívá. tyto parametry nemusejí být dostupné na všech implementacích JAVA. jednoduché ověření lze provézt z příkazové řádky pomocí **java -XX:+UseShenandoahGC -version**. Pokud se zobrazí verze JAVY, můžete následující 4 řádky přidat do nastavení. Pokud se zobrazí chyba o neznámém parametru UseShenandoahGC, pak Vaše implementace JVM tyto parametry nepodporuje.
	
	-XX:+UseShenandoahGC
	-XX:+UnlockExperimentalVMOptions
	-XX:ShenandoahUncommitDelay=10000
	-XX:ShenandoahGuaranteedGCInterval=15000

### Nastavení hesla administrátora  Tomcatu
Pro účely  managerského  přístupu k  Tomcatu  (nahrávání aplikací prostřednictvím webového rozhraní) je třeba do souboru TOMCAT_HOME/conf/tomcat-users.xml doplnit toto:

    <tomcat-users>  
     <role rolename="manager-gui"/>  
       <user password="heslo" roles="manager-gui" username="admin"/>  
    </tomcat-users>


## PostgreSQL
V ABRA BI verze 22.0 není vhodné pro potřeby ukládání dat dál používat embedded databáze H2 nebo HyperSQL. Při vysoké zátěži často dochází k narušení databáze a selhání systému. Doporučuje se  používat PostgreSQL. Embedded databáze zůstávají v  ABRA BI pouze z důvodů zpětné kompatibility.

### Instalace
Instalace do Windows je bezproblémová, stáhnout instalační soubor z adresy:

[https://www.enterprisedb.com/downloads/postgres-postgresql-downloads](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)  ,
Aktuálně je vyzkoušená verze PostGreSQL 14.

při instalaci je nutné definovat port (default je 5432) a vybrat doplňky (nejsou nutné), nezapomenout nainstalovat správcovský vizuální nástroj PGAdmin4, výchozí uživatel je  postgres  s heslem  postgres.
Po ukončení instalace je nutné ručně vytvořit databázi ve vizuálním nástroji PGAdmin4. Doporučený postup je vytvořit oddělenou databázi pro data ABRA BI a druhou, do které budou umístěny snapshot tabulky.


### Nastavení
Doporučuje se v souboru  postgresql.conf  (nachází se na stejném místě jak  hba  soubor - v  datovém adresáři  PostGreSQL) upravit parametr pro používání paměti serverem PostGreSQL.  V základním nastavení je paměť značně malá. Také je třeba zvýšit maximální povolený počet spojení, protože při vysoké zátěži může dočasně dojít k vyčerpání tohoto limitu.

max_connections = 1000
shared_buffers = 2048MB
work_mem = 300MB

**Následně provést restart  PostgreSQL.**

## Nainstalování  abrabi.war
### 1. Aplikace manager/html

Umožňuje jednoduchou administraci aplikačního  serveru - instalaci, spouštění, zastavování a odstraňování aplikací. Vyzkoušení:  
http://localhost:8080/manager/html

### 2. Aplikace Psi-Probe

Lepší prostředí pro administraci aplikačního  serveru - umí  toho víc. Ke stažení je zde:  
[Releases  · psi-probe/psi-probe  · GitHub](https://github.com/psi-probe/psi-probe/releases)

Samotnou instalaci ABRA BI do Tomcatu proveďte buď pomocí manager (http://localhost:8080/manager/html)  nebo pomocí Psi Probe (http://localhost:8080/probe), nebo pomocí prostého nakopírování .war souboru do adresáře TOMCAT_HOME/webapps. Při posledním způsobu instalace aplikací musí soubor .war zůstat v adresáři TOMCAT_HOME/webapps. V případě jeho smazání dojde k odinstalování dané aplikace.

Nyní je ABRA BI dostupná na URl adrese  http://localhost:8080/abrabi

## Následující postupy jsou nepovinné

### Proxy servery 
#### 1. Apache jako  proxy  k  Tomcatu
V případě, že je před  Tomcatem  předsazený  Apache, je potřeba v konfiguraci  Apache  nastavit takto parametr 'ProxyPreserveHost':

    ProxyPreserveHost On

Tím se zajistí, aby  Apache  nastavoval původní  hostname  do atributu 'Host' hlavičky  requestu  redirektovaného  na  Tomcat  (mění tím cílovou URL). Díky tomu pak bude  Wicket  dávat správnou doménu do absolutních URL generovaných dle zadané  page. Pokud se toto  nedodrží, přestane fungovat například přihlašování přes  OAuth  nebo  OpenID  (redirect  zpět z providera).

####  2. Nginx jako  proxy  k  Tomcatu

V případě, že je před  Tomcatem  předsazený  Nginx, je potřeba  Nginx  nakonfigurovat tak, aby URL končící na '/atmo' běžely na  websocket  protokolu.
Konkrétně je potřeba přidat do nastavení  Nginx  definici  **location**  takto (nahradit ADRESA.TOMCATU, parametry $... jsou implicitní):

  

    location ~* ^(.*)/atmo$ {
      # switch off logging
      access_log off;
      proxy_pass  **http://ADRESA.TOMCATU:8080**;
      proxy_redirect  off;
      proxy_set_header  Host  $host;
      proxy_set_header  X-Real-IP  $remote_addr;
      proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
      #Nasledující řádek je potřeba jenom v případě že se definuje serverová část nginx pro HTTPS protokol (a na tomcat se připojuje pomocí HTTP protokolu)
      proxy_set_header  X-Forwarded-Proto https;
      # WebSocket support (nginx 1.4)
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_cache off;
      proxy_request_buffering off;
      }

