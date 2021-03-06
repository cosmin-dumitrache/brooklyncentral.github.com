
        
{% readj ../before-begin.include.md %}

The project ``examples/simple-web-cluster`` includes several deployment descriptors 
for rolling out a web application, under ``src/main/java``.

## Simple Web Server

The simplest of these, ``SingleWebServerExample``, starts JBoss on a single machine with a "Hello World" war deployed,
with a single line:

{% highlight java %}
public class SingleWebServerExample extends AbstractApplication {

    JBoss7Server web = new JBoss7Server(this, 
        war: "classpath://hello-world-webapp.war", 
        httpPort: 8080)
        
    // other housekeeping removed, including public static void main
    // you'll find this in the github repo code mentioned above
     
}
{% endhighlight %}

You can run this (on *nix or Mac) as follows:

{% highlight bash %}
% cd $EXAMPLES_DIR/simple-web-cluster/brooklyn-example-simple-web-cluster/bin
% ./web-cluster.sh
{% endhighlight %}


Then visit the webapp on port 8080, or the Brooklyn console on 8081.  (Default credentials are admin/password.)
Note that the installation may take some time, because the default deployment downloads the software from
the official repos.  You can monitor start-up activity for each entity in the ``Activity`` pane in the management console,
and see more detail by tailing the log file (``tail -f brooklyn.log``).

With appropriate setup (as described [here]({{ site.url }}/use/guide/management/index.html#startup-config)) 
this can also be deployed to your favourite cloud, let's pretend it's Amazon Ireland, as follows: 

{% highlight bash %}
% cd $EXAMPLES_DIR/simple-web-cluster/brooklyn-example-simple-web-cluster
% ./web-server.sh aws-ec2:eu-west-1
{% endhighlight %}


## Elastic Three-Tier

Ready for something more interesting?  Try this:

{% highlight bash %}
% cd $EXAMPLES_DIR/simple-web-cluster/brooklyn-example-simple-web-cluster
% ./web-and-data.sh
{% endhighlight %}

This launches the class ``WebClusterDatabaseExample`` (also described in the [walkthrough]({{ site.url }}/use/walkthrough.html))
which launches a pool of web-servers -- of size 1 initially,
but manually configurable (if you stop the policy first, in the GUI, then use the ``resize`` effector) --
with an Nginx load-balancer set up in front of them, and backed by a MySQL database.

The essential code fragment looks like this:

{% highlight java %}
public class WebClusterDatabaseExample extends AbstractApplication {
    
    ControlledDynamicWebAppCluster web = new ControlledDynamicWebAppCluster(this, war: WAR_PATH);
    MySqlNode mysql = new MySqlNode(this, creationScriptUrl: DB_SETUP_SQL_URL);

    {
        web.factory.configure(
            httpPort: "8080+", 
            (UsesJava.JAVA_OPTIONS):
                ["brooklyn.example.db.url": valueWhenAttributeReady(mysql, MySqlNode.MYSQL_URL,
                    { "jdbc:"+it+"visitors?user=${DB_USERNAME}\\&password=${DB_PASSWORD}" }) ]);

        web.cluster.addPolicy(new
            ResizerPolicy(DynamicWebAppCluster.AVERAGE_REQUESTS_PER_SECOND).
                setSizeRange(1, 5).
                setMetricRange(10, 100));
    }
}
{% endhighlight %}

You can, of course, try this with your favourite cloud, 
tweak the database start script, or drop in your favourite WAR.


## A Few Other Things

The project includes several variants of the examples shown here.
In particular note the pure-Java version in `WebClusterDatabaseExampleAltJava.java`
some other variations in syntax (the `*Alt*` files), and a
web-only cluster (no database) in ``WebClusterExample``.

The webapp that is used is included under ``examples/hello-world-webapp``.

You may wish to check out the [Global Web Fabric example]({{ site.url }}/use/examples/global-web-fabric/) next.

If you encounter any difficulties, please [tell us]({{ site.url }}/meta/contact.html) and we'll do our best to help.
