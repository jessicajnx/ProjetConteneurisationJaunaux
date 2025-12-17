Build de l'image:



docker build -t linuxmint75/rentalservice:1.0 .





Lancement du container:



PS C:\\Users\\Jess\\Documents\\ingnum\\RentalService> docker run linuxmint75/rentalservice:1.0 -p 8080:8080

OU

PS C:\\Users\\Jess\\Documents\\ingnum\\RentalService> docker run linuxmint75/rentalservice:1.0 -P 

car EXPOSE est déjà défini dans le dockerfile mais cela ménera au binding vers un port aléatoire



&nbsp; .   \_\_\_\_          \_            \_\_ \_ \_

&nbsp;/\\\\ / \_\_\_'\_ \_\_ \_ \_(\_)\_ \_\_  \_\_ \_ \\ \\ \\ \\

( ( )\\\_\_\_ | '\_ | '\_| | '\_ \\/ \_` | \\ \\ \\ \\

&nbsp;\\\\/  \_\_\_)| |\_)| | | | | || (\_| |  ) ) ) )

&nbsp; '  |\_\_\_\_| .\_\_|\_| |\_|\_| |\_\\\_\_, | / / / /

&nbsp;=========|\_|==============|\_\_\_/=/\_/\_/\_/

&nbsp;:: Spring Boot ::                (v3.2.1)



2025-12-17T16:20:59.374Z  INFO 1 --- \[RentalService] \[           main] c.i.r.RentalServiceApplication           : Starting RentalServiceApplication using Java 21.0.9 with PID 1 (/app.jar started by root in /)

2025-12-17T16:20:59.381Z  INFO 1 --- \[RentalService] \[           main] c.i.r.RentalServiceApplication           : No active profile set, falling back to 1 default profile: "default"

2025-12-17T16:21:01.362Z  INFO 1 --- \[RentalService] \[           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)

2025-12-17T16:21:01.391Z  INFO 1 --- \[RentalService] \[           main] o.apache.catalina.core.StandardService   : Starting service \[Tomcat]

2025-12-17T16:21:01.392Z  INFO 1 --- \[RentalService] \[           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: \[Apache Tomcat/10.1.17]

2025-12-17T16:21:01.469Z  INFO 1 --- \[RentalService] \[           main] o.a.c.c.C.\[Tomcat].\[localhost].\[/]       : Initializing Spring embedded WebApplicationContext

2025-12-17T16:21:01.474Z  INFO 1 --- \[RentalService] \[           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1903 ms

2025-12-17T16:21:02.175Z  INFO 1 --- \[RentalService] \[           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path ''

2025-12-17T16:21:02.206Z  INFO 1 --- \[RentalService] \[           main] c.i.r.RentalServiceApplication           : Started RentalServiceApplication in 3.825 seconds (process running for 4.958)





Push de l'image sur le repository:

docker push linuxmint75/rentalservice:1.0



Pour récupérer l'image

docker pull linuxmint75/rentalservice:1.0

