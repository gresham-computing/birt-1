# BIRT
The open source Eclipse BIRT reporting and data visualization project. 

## Compiling 🤞
Firstly, good luck. The branch `BIRT_4_5_Branch` which we use is old and some of the dependencies are no longer available on the internet, so we are only able to compile some projects rather than BIRT as a whole.

| Project | Compiles? |
| --- | :---: |
| uk.co.spudsoft.birt.emitters.excel | ✔ |
| org.eclipse.birt.report.engine | ✖ |

## BIRT artifacts used in CTC 🤯
There are 2 artifacts in the reports.pu. 

The _runtime_ contains the actual java classes with any of our tweaks, such as changes to `uk.co.spudsoft.birt.emitters.excel`.
```
<groupId>com.gresham</groupId>
<artifactId>org.eclipse.birt.runtime</artifactId>
```

The _war_ is the web application that contains the runtime jar described above, amongst all the other BIRT jars.
```
<groupId>com.gresham</groupId>
<artifactId>birt.war</artifactId>
```

## Testing changes in CTC 💥
It is quick to make a change in BIRT and get it into a running CTC reports PU.
1. Compile changes for `uk.co.spudsoft.birt.emitters.excel` (or your favourite project you are working in)
1. Find your modified classes in `uk.co.spudsoft.birt.emitters.excel/target/`
1. Find `ctcserver/xap/gigaspaces-xap-premium-12.3.0-ga/deploy/reports/WEB-INF/lib/org.eclipse.birt.runtime-4.5.0-10-SNAPSHOT.jar` (actual version may vary)
1. Copy the modified classes into the jar
1. Restart the reports PU (takes 30-60 seconds).

## Preparing and Uploading to Nexus 📦
Once happy with the changes they can be uploaded to Nexus. This is done by modifying the latest runtime jar and birt war and uploading with a new version. 

1. Download the latest release version of these artifacts from the `thirdparty-maven-releases` Nexus repo
1. Rename the jar and war to the next snapshot
1. Similar to the testing steps above, copy the modified classes into the runtime jar
1. Replace the modified runtime jar within the birt war `/WEB-INF/lib/` directory.
1. The 2 artifacts can be uploaded in either order. The properties you need to think about are:
    - `version`: the next snapshot / release
    - `url`: the nexus url to either `thirdparty-maven-snapshots` or `thirdparty-maven-releases`
    - `repositoryId`: this is a reference to credentials in your maven `settings.xml`. The value should probably be `snapshots` or `releases`.
    
    ```
    mvn deploy:deploy-file -DgroupId=com.gresham -DartifactId=org.eclipse.birt.runtime -Dversion=4.5.0-10-SNAPSHOT -Dpackaging=jar -Dfile=org.eclipse.birt.runtime-4.5.0-10-SNAPSHOT.jar -DrepositoryId=snapshots -Durl=https://nexus.greshamtech.com/repository/thirdparty-maven-snapshots/
    
    mvn deploy:deploy-file -DgroupId=com.gresham -DartifactId=birt.war -Dversion=4.5.0-10-SNAPSHOT -Dpackaging=war -Dfile=birt.war-4.5.0-10-SNAPSHOT.war -DrepositoryId=snapshots -Durl=https://nexus.greshamtech.com/repository/thirdparty-maven-snapshots/
    ```

## Referencing new artifacts in CTC 🔢
In the CTC server.parent pom, change the `birt.version` property to the version of the new artifacts. A `mvn clean install` on the reports.pu project will create a new war, which can be copied and pasted into a distribution or running server. Maybe even consider running the tests such as `BirtReportsEngineTest`.

## Releasing 🎉
1. Rename your jar and war to a `.RELEASE`
1. Put the release jar into the war and remove the snapshot jar
1. Upload artifacts to Nexus
1. Update the artifact version in the server.parent pom.

Congratulations! You deserve a trip on The Matthew for getting this far.

## Gotchas 🤔
- BIRT is OSGi based and therefore more strict with version syntax. For example a version of `4.5.0-10-SNAPSHOT` may not work as it has more than 1 dash. or at least that is my theory as to why `BirtReportsEngineTest` within CTC is failing.
