# OpenShift Spring Boot Example
This is a very basic [Spring Boot](http://projects.spring.io/spring-boot/) app using Java 8, built with [Gradle](http://gradle.org/), and hosted on [OpenShift](https://www.openshift.com/).


## How To Use

First off sign-up for a free [OpenShift](https://www.openshift.com/) account and [install the client tools](https://developers.openshift.com/managing-your-applications/client-tools.html).

Next, create a new app using the DIY cartridge (this will create the app and clone the repository to *<app-name>* directory):

    rhc app create <app-name> diy-0.1

Delete the template project:

    cd <app-name>
    git rm -rf .openshift README.md diy misc
    git commit -am "Remove template application source code"

Pull in this repository:

    git remote add upstream https://github.com/ukdave/openshift-spring-boot-gradle.git
    git pull -s recursive -X theirs upstream master

Finally, push to OpenShift:

    git push

The initial deployment may take a couple of minutes while it downloads Gradle and builds the code. Subsequent deployments will be faster.

If everything worked you should now be able to browse to `https://<app-name>-<domain>.rhcloud.com/` and you should see "Hello World". Management information (courtesy of Spring Boot Actuator) is available at `https://<app-name>-<domain>.rhcloud.com/manage/`

You can also tail the application log file with:

    rhc tail <app-name>


## Details

This is just a standard Spring Boot app (although I've chosen to use Undertow instead of Tomcat). The only bits needed to get it running on OpenShift are in the `.openshift` directory:

- `deploy` - OpenShift runs this script whenever you push the repository. It installs Gradle if not already installed, then cleans and builds the code. This will create a single fat-JAR under the `build` directory.
- `start` - OpenShift runs this to start the app. It sets the PATH to point to Java 8 and then runs `java -jar` to launch the Spring Boot app.
- `stop` - OpenShift runs this to stop the app. It uses `ps` to find the PID of the Spring Boot app and then uses `kill` to stop it.

The `start` script also sets the active Spring profile to one called `openshift`. This allows us to configure any application properties specific to the OpenShift environemnt (e.g. database connections) in `application-openshift.yml` overriding any settings for local development in `application.yml`.
