# Tech Document of *[Group 2  Web Page](https://www.geos.ed.ac.uk/~s2298227/CGS/)*
## 1. Introduction
### 1.1 Framework
#### 1.1.1 HTML
In order to create a cross-platform, multi-device web page as efficiently as possible, this website uses the HTML template from [html5up.net](https://www.html5up.net) by @ajlkn.  
>Usage under specific [License](https://www.geos.ed.ac.uk/~s2298227/CGS/LICENSE.txt).
#### 1.1.2 JavaScript
Along with the HTML template, this website uses many third-part javascript plugins to achieve several helpful functions. All plugins not include in the template are listed below.
- [Leaflet](https://leafletjs.com/)
- [Leaflet.legend](https://github.com/ptma/Leaflet.Legend)
- [echarts](https://echarts.apache.org/)
#### 1.1.3 Cloud Storage
Thanks to [CloudFlare](https://www.cloudflare.com/) `R2 Bucket`, data can be easily stored and fetched. In this project, all `geoJSON` files are uploaded to `R2 Bucket` for further usage.
#### 1.1.4 Serverless API
##### 1.1.4.1 LikeThis API
As an important way of representing likeness of this website, a function of thumbs-up is developed with the support of `Worker`
feature by [CloudFlare](https://www.cloudflare.com/). All requests are restricted to the [specific domain](https://www.geos.ed.ac.uk).
```JavaScript
//Likethis
export default {
    async fetch(request, env) {
        switch (request.method) {
            case 'GET':
                let obj = await env.views.get('likes');
                if (obj === null) {
                    return new Response('Request Failed', {status: 404});
                }
                const url = new URL(request.url);
                const key = url.pathname.slice(1);
                if (key === 'like'){
                    await env.views.put('likes',(Number(obj)+1));
                }
                let headers = new Headers();
                headers.set("Access-Control-Allow-Origin", "https://www.geos.ed.ac.uk");
                headers.set("Access-Control-Allow-Methods", "GET");
                return new Response(obj, {headers,});
            default:
                return new Response('Method Not Allowed', {status: 405, headers: {Allow: 'PUT, GET',},});
        }
    },
};
```
##### 1.1.4.2 GetData API
To avoid unauthentic request of cloud storage, this API is developed to get specific data from `R2 Bucket`. All requests are restricted to the [specific domain](https://www.geos.ed.ac.uk).
```JavaScript
//Get R2 Data
export default {
    async fetch(request, env) {
        const url = new URL(request.url);
        const key = url.pathname.slice(1);
        switch (request.method) {
            case 'GET':
                const object = await env.data.get(key);
                if (object === null) {
                    return new Response('Object Not Found', {status: 404});
                }
                const headers = new Headers();
                headers.set("Access-Control-Allow-Origin", "https://www.geos.ed.ac.uk");
                headers.set("Access-Control-Allow-Methods", "GET");
                object.writeHttpMetadata(headers);
               
                return new Response(object.body, {headers,});
            default:
                return new Response('Method Not Allowed', {status: 405, headers: {Allow: 'PUT, GET, DELETE',},});
        }
    },
}
```
#### 1.1.5 Database
With the support of [School of GeoScience (UoE)](https://www.geos.ed.ac.uk), this project has the access to the school's domain and Oracle Database.
We design the database as a part of our project and provide a sample usage on the [website](https://www.geos.ed.ac.uk/~s2298227/CGS/index.html) (Map Layers->Diagrams).
#### 1.1.6 Python
This project uses `Python` to interact with the database and runs on school's  `Linux` server.All related modules are listed below.
- [cgitb](https://github.com/python/cpython/blob/3.11/Lib/cgitb.py)
- [cx_Oracle](https://oracle.github.io/python-cx_Oracle/)
- [jinja2](https://palletsprojects.com/p/jinja/)


## 2 Detailed Development
This section includes the key structure of each code file. 
>**For more information, Please refer to the Source Code in Appendix.**
### 2.1 Main Page [Source Code](#a1-indexhtml)
#### 2.1.1 Layout
The whole page consists of three main parts, namely the navigation bar, the sidebar, and the content body.  
In the Navigation Bar, **Intro**, **Map Layer**, **Project Report**, **About Us** and **References** are set up according to the project content to provide map layers, a preview and download of the report, an introduction to the project members and relevant references respectively.  
The Sidebar shows the topic of the project, research content, team members and acknowledgements.  
The Content Body is made up of several elements, corresponding to the tabs in the navigation bar, which are used to present different aspects of the content without jumping through multiple pages.    

*PS:We have also added the University of Edinburgh icon to the site title.*  
#### 2.1.2 Tab Switch
To enable separate content to be displayed without jumping through multiple pages, a function has been written to change the display properties of different elements on this page.
```JavaScript
let i;
let as = document.querySelectorAll("#tab a")
for (i = 0; i < as.length; i++) {
    as[i].onclick = function () {
        let divs = document.querySelectorAll("#content>div");
        for (i = 0; i < divs.length; i++) {
            divs[i].style.display = "none";
        }
        let div_current = this.href.slice(this.href.lastIndexOf('#'));
        document.querySelector(div_current).style.display = "inline";
    }
}
```
#### 2.1.3 LikeThis
As mentioned in [LikeThis API](#1141-likethis-api) the framework section, this page implements a very interesting little feature to help viewers easily express their love for the project.
```javascript
function get_thumb() {
    function get_data(callback) {
        jQuery.get('https://likethis.annierzhy.workers.dev/').done(function (data) {
            let likes = data
            callback(likes);
        });
    }
    get_data(function (data) {
        document.getElementById('tag').innerHTML = data;
    })
}
function thumbs_up() {
    jQuery.get('https://likethis.annierzhy.workers.dev/like').done(function (){
        get_thumb();
    });

}
```
#### 2.1.4 iframe
This page contains several iframes for nesting different content.
```java
//Map Layer Content
<iframe src="layer.html" width="100%" height="700px"></iframe>
//Diagram of Neibourhoods
<iframe src="https://www.geos.ed.ac.uk/~s2298227/cgi-bin/nei.py" width="800px" height="600px"></iframe>
//Diagram of Species
<iframe src="https://www.geos.ed.ac.uk/~s2298227/cgi-bin/spec.py" width="800px" height="600px"></iframe>
//Preview of report
<iframe src="files/Report.pdf" width="100%" height="700px"></iframe>
```
### 2.2 Map Layer [Source Code](#a2-layerhtml)
#### 2.2.1 Layout
This page mainly shows the entire map layers.  
The control and selection of the different base and overlay layers is implemented in the top right corner of the map, while the legend and the dynamic scale are shown in the bottom left corner respectively.  
The base map can be switched and the overlay layer switched on and off by clicking on it.
#### 2.2.2 Load Data
As stated in the [GetData API]((#1142-getdata-api)) of framework section, all of the overlay layer data is stored in [cloud storage](#113-cloud-storage) and this page needs to implement the loading of the data.
```javascript
jQuery.getJSON('https://cgsdata.annierzhy.workers.dev/Nei.json', function (data) {
    cur = 'Nei';
    L.geoJSON(data, {
            onEachFeature: onEachFeature,
        }
    ).addTo(Nei);
    return true
});
```
#### 2.2.3 Feature Style
Functions have been written to differentiate between layers and to implement the relevant functions on the corresponding layers.
```javascript
function onEachFeature(feature, layer) {
    if (cur === 'Carbon') {
        layer.on(
            mouseover: (e) => {
                let layer = e.target;
                layer.setStyle({
                    weight: 5,
                    color: 'red',
                    dashArray: '',
                    fillOpacity: 0.7
                })
                e.target.openPopup();
            },
            mouseout: (e) => {
                e.target.closePopup();
                layer.setStyle({
                    color: 'rgb(198,252,198)',
                    fillOpacity: 1,
                })
            }
        )   
            layer.bindPopup("Neighbourhood: " + feature.properties.LABEL + "</br>" + ("Tons of CO&#8322; eq:" + feature.properties.TonCO2eq).slice(0, 29)).openPopup();
    }
}
```
### 2.3 Diagrams
#### 2.3.1 HTML
These files contain the html templates of presenting data in charts.  
>See [Source Code](#c1-neihtml) for more details.
#### 2.3.2 Python
These files implement the function of getting the relevant contents in the database and pass them to the templates.  
>See [Source Code](#c3-neipy) for more details.
### 2.4 Database Scripts
#### 2.4.1 NEI.sql & SPEC.sql

These files implement the relevant database table creation operations.
>See [Source Code](#d1-neisql) for more details.

#### 2.4.2 NEI_LDR.ctl & SPEC_LDR.ctl
These files implement the relevant data import operations into the Database.
>See [Source Code](#d3-nei_ldrctl) for more details.

### 2.5 Manuals
This part contains the manual for the website, which is implemented through the use of HTML templates, CSS and JS.
>See [Source Code](#appendix-d-manual-pages) for more details.

## 3 Acknowledgement and References
### 3.1 Acknowledgement
Would like to give special thanks to our teachers **Dr. Bruce Gittings**, **Dr. Neil Stuart**, **Dr. Zhiqiang Feng** and **Dr. Kathryn** Murphy for their guidance throughout this Capital Green Space Project.

Also, would like to give thanks to **Professor Iain Woodhouse** (University of Edinburgh), **Dr. Emil Cherrigton** (GEE Expert,University of Alabama, NASA SERVIR), **Dr. Percival Cho** (Forestry Expert of Belize), **Florencia Gurrea** (Forest Officer, Belize), **Eduardo Reyes** (Senior Climate Change Expert, Coalition of Rainforest Nations) and **Kieron Doick** (Forest Research UK).

Having guidance from all these experts allowed us to provide a successful report on the value of Carbon within the greenspace of Edinburgh.  

>WITH SPECIAL THANKS TO ORGANIZATIONS
>- [ESA](https://esa-worldcover.org/)
>- [ESRI](https://www.esri.com/)
>- [Forest Commission](https://www.gov.uk/government/organisations/forestry-commission)
>- [Forest Resaerch](https://www.forestresearch.gov.uk/)
>- [Scottish Index of Multiple Deprivation(SIMD)](https://simd.scot/)
>- [Ordance Survey](https://www.ordnancesurvey.co.uk/)
### 3.2 References
>See [Website](https://www.geos.ed.ac.uk/~s2298227/CGS/) for more details.

# Appendix
## Appendix A Main Web HTML
### A.1 index.html
```html
<!DOCTYPE HTML>
<html lang="en">
<head>
    <title>Capital Green Space Project - Value in terms of a carbon sink</title>
    <link rel="icon" type="image/png" sizes="32x32" href="https://www.ed.ac.uk/sites/all/themes/uoe/favicon/favicon-32x32.png?t=rlhfo6" />
    <link rel="icon" type="image/png" sizes="16x16" href="https://www.ed.ac.uk/sites/all/themes/uoe/favicon/favicon-16x16.png?t=rlhfo6" />
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no"/>
    <link rel="stylesheet" href="assets/css/main.css"/>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.3/dist/leaflet.css"
          integrity="sha256-kLaT2GOSpHechhsozzB+flnD+zUyjE2LlfWPgU04xyI=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.3/dist/leaflet.js"
            integrity="sha256-WBkoXOwTeyKclOHuWtc+i2uENFpDZ9YPdf5Hf+D7ewM=" crossorigin=""></script>
    <link rel="stylesheet" href="assets/css/leaflet.legend.css"/>
    <script type="text/javascript" src="assets/js/leaflet.legend.js"></script>
</head>
<body class="is-preload">
<div id="wrapper">
    <!-- Header-->
    <header id="header">
        <h1><a href="">Value in terms of a carbon sink</a></h1>
        <nav class="links">
            <ul id="tab">
                <li><a href="#page_intro">Intro</a></li>
                <li><a href="#page_map">Map Layers</a></li>
                <li><a href="#page_report">Project Report</a></li>
                <li><a href="#page_about">About Us</a></li>
                <li><a href="#page_refer">References</a></li>
            </ul>
        </nav>
        <nav class="main">
            <ul>
                <li class="menu">
                    <a class="fa-bars" href="#menu">Menu</a>
                </li>
            </ul>
        </nav>
    </header>
    <!-- Menu-->
    <section id="menu">
        <!--    SideBar-->
        <section>
            <ul class="links" id="side_tab">
                <li>
                    <a href="#page_map">
                        <h3>Map Layers</h3>
                        <p>Vivid View of Project Results</p>
                    </a>
                </li>
                <li>
                    <a href="#page_report">
                        <h3>Project Report</h3>
                        <p>PDF Version of Project Report</p>
                    </a>
                </li>
                <li>
                    <a href="#page_about">
                        <h3>About Us</h3>
                        <p>An Introduction to Our Team Members</p>
                    </a>
                </li>
                <li>
                    <a href="#page_refer">
                        <h3>References</h3>
                        <p>All related References</p>
                    </a>
                </li>
            </ul>
        </section>
    </section>
    <!--All Pages-->
    <div id="content">
        <!--Introduction Page-->
        <div id="page_intro">
            <article class="post">
                <header>
                    <div class="title">
                        <h2><a href="#">Introduction</a></h2>
                        <p>What we have done and how we did it.</p>
                    </div>
                    <div class="meta">
                        <time class="published" datetime="2023-01-01">January, 2023</time>
                        <a href="#" class="author"><span class="name">Group 2</span></a>
                    </div>
                </header>
                <span><img src="images/scope.jpg" alt="" width="100%"/></span>
                <p>Vegetation contained within green space is to improve air quality and alleviate climate change
                    through carbon sequestration. Green spaces are classified as woodlands, which classifies trees based
                    on leaf types and growing status and non-woodlands, which refers to urban vegetation and grassland
                    (Forestry Commission, 2001). The research carried out in this project is based on the Central
                    Scotland Green Network (CSGN) objectives of extending and improving the quality of green spaces
                    (CSGN and Green Action Trust, 2020).</p>
                <p>This study aims to define, classify and identify Edinburgh’s green spaces to determine their value as
                    a carbon sink. The ability of trees to sequester harmful greenhouse gases such as CO2 through
                    photosynthesis highlights their essential contributions to decarbonisation goals such as Edinburgh’s
                    target of achieving net-zero by 2030 (Williamson et al., 2020). In addition, to being a ‘climate
                    resilient city’ (The City of Edinburgh Council, 2021) quality greens paces in Edinburgh connect
                    neighbourhoods and enrich the lives of local communities through the provision of recreational
                    surfaces (Singkran, 2022). Green space management is therefore essential in sustaining a productive
                    urban ecosystem, which contributes to both the mitigation of climate change and environmental
                    deprivation. It is evident that Edinburgh’s green spaces have a significant community and
                    environmental value; however, this study will investigate the latter with specific reference to the
                    value of carbon contained within green spaces. Since 50% of dry biomass is made up of carbon,
                    biomass estimations can be used to determine carbon storage within Edinburgh’s green space (Kumar
                    and Mutanga, 2019).</p>
                <footer>
                    <ul class="stats">
                        <button id="like" onclick="thumbs_up()">Like This</button>
                        <li><a class="icon solid fa-heart" id="tag">0</a></li>
                    </ul>
                </footer>
            </article>
            <article class="post">
                <header>
                    <div class="title">
                        <h2><a href="#">Discussions</a></h2>
                    </div>
                    <div class="meta">
                        <time class="published" datetime="2023-01-01">January, 2023</time>
                        <a href="#" class="author"><span class="name">Group 2</span></a>
                    </div>
                </header>
                <li style="font-size: x-large"><strong>Estimation of Above-Ground Biomass Carbon in Green
                    Spaces</strong></li>
                <p>A combination of national and global data sources was obtained to estimate and define green spaces
                    for the scope of the project. This was done through a literature review and expert opinions.
                    Furthermore, with spatial analysis tools in ArcGIS Pro, a total of 3,679 ha of green spaces was
                    estimated to include Woodland (Broadleaved and Conifers) and Non-woodlands (Urban Forest Areas). See
                    the Map Below. </p>
                <span><img src="images/DS2.png" alt="" width="100%"/></span>
                <p>
                    Knowing the breakdown and the area for green spaces, emission factor information was obtained from
                    the Intergovernmental Panel on Climate Change (IPCC), Forest Lands Chapter 4 Report, to apply the
                    equation for calculating Biomass Carbon Stock Tonnes per year. Below is a table and description of
                    how the equation was applied. The total sum that was estimated for green spaces in the study site
                    was approximately 261,622 tons of CO2 per year. For more information and details you can access the
                    full report <a href="files/Report.pdf">here</a> .
                </p>
                <span><img src="images/DS3.png" alt="" width="100%"/></span>
                <li style="font-size: x-large"><strong>Environmental deprivation and management strategies </strong>
                </li>
                <span><img src="images/RF3.png" alt="" width="100%"/></span>
                <p>This project has identified areas that could be viable for tree planting in both public and private
                    green spaces to increase carbon storage. City Council is encouraged to pursue targeted management in
                    these areas through City Council work in public spaces as well as schemes to encourage planting in
                    private spaces to be able to reach the ‘one million tree city by 2030’ objective that connects to
                    the overall net zero by 2030 goal for the city of Edinburgh. </p>
                <footer>
                </footer>
            </article>
        </div>
        <!--Map Page-->
        <div id="page_map">
            <!--Map-->
            <article class="post">
                <header>
                    <div class="title">
                        <h2>Green Space in Edinburgh</h2>
                        <p>How much carbon they stored</p>
                    </div>
                    <div class="meta">
                        <time class="published" datetime="2023-01-01">January, 2023</time>
                        <a href="#" class="author"><span class="name">Group 2</span></a>
                    </div>
                </header>

                <iframe src="layer.html" width="100%" height="700px"></iframe>

                <footer>
                </footer>
            </article>
            <!--Diagrams-->
            <article class="post">
                <header>
                    <div class="title">
                        <h2>Carbon Storage Per Neighbourhood</h2>
                    </div>
                </header>
                <iframe src="https://www.geos.ed.ac.uk/~s2298227/cgi-bin/nei.py" width="800px" height="600px"></iframe>
            </article>
            <article class="post">
                <header>
                    <div class="title">
                        <h2>Carbon Storage Per Specie</h2>
                    </div>
                </header>
                <iframe src="https://www.geos.ed.ac.uk/~s2298227/cgi-bin/spec.py" width="800px" height="600px"></iframe>
            </article>

        </div>
        <!--Report Page-->
        <div id="page_report">

            <article class="post">
                <header>
                    <div class="title">
                        <h2><a href="#">Project Report</a></h2>
                        <p>Detailed Report Document.</p>
                    </div>
                    <div class="meta">
                        <time class="published" datetime="2023-01-01">January, 2023</time>
                        <a href="#" class="author"><span class="name">Group 2</span></a>
                    </div>
                </header>
                <span class="image featured"><img src="images/report.png" alt=""/></span>

                <iframe src="files/Report.pdf" width="100%" height="700px"></iframe>
                <footer>
                </footer>
            </article>

        </div>
        <!--About Page-->
        <div id="page_about">
            <article class="post">
                <header>
                    <div class="title">
                        <h2><a href="#">About us</a></h2>
                        <p>Who we are and what we do.</p>
                    </div>
                    <div class="meta">
                        <time class="published" datetime="2023-01-01">January, 2023</time>
                        <a href="#" class="author"><span class="name">Group 2</span></a>
                    </div>
                </header>
                <span class="image featured"><img src="images/About.png" alt=""/></span>
                <h1>Get In Touch</h1>
                <li><a href="https://www.linkedin.com/in/edgar-correa-b8004581"><img src="images/linkedin.png"
                                                                                     width="20"
                                                                                     height="20" alt="Linkedin"></a>
                    <strong> Edgar Correa</strong> <a href="mailto:s2270067@ed.ac.uk" class="icon solid fa-envelope">
                        <span class="label">Email</span></a>
                </li>
                <li><img src="images/linkedin.png" width="20" height="20" alt="Linkedin">
                    <strong> Haocheng Cai</strong> <a href="mailto:s2440837@ed.ac.uk" class="icon solid fa-envelope">
                        <span class="label">Email</span></a>
                </li>
                <li><a href="https://www.linkedin.com/in/%E6%B5%A9%E5%AE%87-%E5%BC%A0-582158116/?locale=en_US"><img
                        src="images/linkedin.png"
                        width="20"
                        height="20" alt="Linkedin"></a>
                    <strong> Haoyu Zhang</strong> <a href="mailto:s2298227@ed.ac.uk" class="icon solid fa-envelope">
                        <span class="label">Email</span></a>
                </li>
                <li><img src="images/linkedin.png" width="20" height="20" alt="Linkedin">
                    <strong> Kate Rollinson</strong> <a href="mailto:s2270131@ed.ac.uk" class="icon solid fa-envelope">
                        <span class="label">Email</span></a>
                </li>
                <li><img src="images/linkedin.png" width="20" height="20" alt="Linkedin">
                    <strong> Ruoxing Lan</strong> <a href="mailto:s2283650@ed.ac.uk" class="icon solid fa-envelope">
                        <span class="label">Email</span></a>
                </li>
                <li><a href="https://www.linkedin.com/in/stephanie-long-ab1051168"><img src="images/linkedin.png"
                                                                                        width="20"
                                                                                        height="20" alt="Linkedin"></a>
                    <strong> Stephanie Long</strong> <a href="mailto:s2413287@ed.ac.uk" class="icon solid fa-envelope">
                        <span class="label">Email</span></a>
                </li>
                <footer>
                </footer>
            </article>
        </div>
        <!--Reference Page-->
        <div id="page_refer">
            <article class="post">
                <header>
                    <div class="title">
                        <h2><a href="#">References</a></h2>
                        <p>All Related References.</p>
                    </div>
                    <div class="meta">
                        <time class="published" datetime="2023-01-01">January, 2023</time>
                        <a href="#" class="author"><span class="name">Group 2</span></a>
                    </div>
                </header>
                <p><a href="files/RF1.pdf">1.Central Scotland Green Network Delivery Plan 2020-2030</a></p>
                <img src="images/RF1.png" alt="" height="500px"/>
                <p><a href="files/RF2.pdf">2.The Reporting for Results-Based REDD+ Project (RRR+)</a></p>
                <img src="images/RF2.png" alt="" height="500px"/>
                <p><a href="files/RF3.pdf">3.2030 Climate Strategy</a></p>
                <img src="images/RF3.png" alt="" width="100%"/>
                <p><a href="files/RF4.pdf">4.Valuing Edinburgh’s Urban Trees</a></p>
                <img src="images/RF4.png" alt="" height="500px"/>
                <p><a href="files/RF5.pdf">5.National Inventory of Woodland and Trees</a></p>
                <img src="images/RF5.png" alt="" height="500px"/>
                <p><a href="files/RF6.pdf">6.Technical Specification for the Biomass Equations Developed for the 2011 Forecast</a>
                </p>
                <img src="images/RF6.png" alt="" height="500px"/>
                <p><a href="files/RF7.pdf">7.The Scottish Index of Multiple Deprivation 2020</a></p>
                <img src="images/RF7.png" alt="" height="500px"/>
                <p><a href="files/RF8.pdf">8.ESA World Cover - Product User Manual</a></p>
                <img src="images/RF8.png" alt="" height="500px"/>
                <p><a href="files/RF9.pdf">9.OS MASTERMAP GREENSPACE LAYER™ – TECHNICAL SPECIFICATION</a></p>
                <img src="images/RF9.png" alt="" height="500px"/>
                <footer>
                </footer>
            </article>
        </div>

    </div>

    <section id="sidebar">

        <section id="intro">
            <a href="#" class="logo"><img src="images/logo.jpg" alt=""/></a>
            <header>
                <h2>Capital Green Space Project</h2>
                <p>Value in Terms of a Carbon Sink</p>
            </header>
        </section>

        <section class="blurb">
            <li style="font-size: large">Have questions while visit this website?</br>
            <a href="manual/homepage.html"><strong>Read User Guide Here!</strong></a></li></br>
            <h2>Group Members:</h2>
            <p>Edgar Correa</br>  Haocheng Cai</br>  Haoyu Zhang</br>  Kate Rollinson</br>  Ruoxing Lan</br>  Stephanie
                Long</p>
            <ul class="actions">
                <li><a href="https://arcg.is/1qaLK2" class="button">Learn More</a></li>
            </ul>
        </section>

        <section id="footer">
            <h1>Acknowledgments:</h1>
            <p>Would like to give special thanks to our teachers <strong>Dr. Bruce Gittings</strong>, <strong>Dr. Neil
                Stuart</strong>, <strong>Dr. Zhiqiang Feng</strong> and <strong>Dr. Kathryn Murphy</strong> for their
                guidance throughout this Capital Green Space Project.
            </p>
            <p>Also, would like to give thanks to <strong>Professor Iain Woodhouse</strong> (University of Edinburgh),
                <strong>Dr. Emil Cherrigton</strong> (GEE Expert,University of Alabama, NASA SERVIR), <strong>Dr.
                    Percival Cho</strong> (Forestry Expert of Belize),
                <strong>Florencia Gurrea</strong> (Forest Officer, Belize), <strong>Eduardo Reyes</strong> (Senior
                Climate Change Expert, Coalition of
                Rainforest Nations) and <strong>Kieron Doick</strong> (Forest Research UK).</p>
            <p>Having guidance from all these experts allowed us to provide a successful report on the value of Carbon
                within the greenspace of Edinburgh.</p>
            <ul class="icons">
                <h1>With Special Thanks To:</h1>
                <li><img src="images/ForestCommission.png" width="150px" alt="ForestCommission"></li>
                <li><img src="images/esri.png" width="150px" alt="ESRI"></li>
                <li><img src="images/simd.png" width="150px" alt="SIMD"></li>
                <li><img src="images/esa.png" width="150px" alt="ESA"></li>
                <li><img src="images/ForestResearch.png" width="150px" alt="ForestResearch"></li>
                <li><img src="images/os.png" width="100px"></li>


            </ul>
            <p class="copyright">&copy;University of Edinburgh</br><a href="https://www.geos.ed.ac.uk"></a>School of
                GeoScience </p>
            <p class="copyright"><a href="LICENSE.txt">License</a> </p>
        </section>

    </section>

</div>
</body>
<script src="assets/js/jquery.min.js"></script>
<script src="assets/js/browser.min.js"></script>
<script src="assets/js/breakpoints.min.js"></script>
<script src="assets/js/util.js"></script>
<script src="assets/js/main.js"></script>
<script>
    // ThumbsUp Function
    function get_thumb() {
        function get_data(callback) {
            jQuery.get('https://likethis.annierzhy.workers.dev/').done(function (data) {
                let likes = data
                callback(likes);
            });
        }
        get_data(function (data) {
            document.getElementById('tag').innerHTML = data;
        })
    }
    function thumbs_up() {
        jQuery.get('https://likethis.annierzhy.workers.dev/like').done(function (){
            get_thumb();
        });

    }
    // Navigate Bar Function
    window.onload = function () {
        //Initialize Thumbs-up Functions
        get_thumb();
        // Navigation Bar Control
        let i;
        let as = document.querySelectorAll("#tab a")
        for (i = 0; i < as.length; i++) {
            as[i].onclick = function () {
                let divs = document.querySelectorAll("#content>div");
                for (i = 0; i < divs.length; i++) {
                    divs[i].style.display = "none";
                }
                let div_current = this.href.slice(this.href.lastIndexOf('#'));
                document.querySelector(div_current).style.display = "inline";
            }
        }
        // Side Bar Control
        let sb = document.querySelectorAll("#side_tab a")
        for (i = 0; i < as.length; i++) {
            if (sb[i]) {
                sb[i].onclick = function () {
                    let divs = document.querySelectorAll("#content>div");
                    for (i = 0; i < divs.length; i++) {
                        divs[i].style.display = "none";
                    }
                    let div_current = this.href.slice(this.href.lastIndexOf('#'));
                    document.querySelector(div_current).style.display = "inline";
                }
            }
        }
    };
</script>
</html>
```

### A.2 layer.html
```html
<!DOCTYPE HTML>
<html lang="en">
<head>
    <meta charset="utf-8"/>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.3/dist/leaflet.css"
          integrity="sha256-kLaT2GOSpHechhsozzB+flnD+zUyjE2LlfWPgU04xyI=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.3/dist/leaflet.js"
            integrity="sha256-WBkoXOwTeyKclOHuWtc+i2uENFpDZ9YPdf5Hf+D7ewM=" crossorigin=""></script>
    <link rel="stylesheet" href="assets/css/leaflet.legend.css"/>
    <script type="text/javascript" src="assets/js/leaflet.legend.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
</head>
<body>
<!--Set Map Element-->
<div id='map' style="width: 100%; height: 600px;"></div>
</body>
<script>
    function onEachFeature(feature, layer) {
        // Set style of Natural Neighbourhood Layer
        if (cur === 'Nei') {
            layer.bindPopup(feature.properties.LABEL).openPopup();
            layer.setStyle({
                color: '#000000',
                fillOpacity: 0,
                weight: 1,
            })
        }
        // Set style of SIMD Layer
        if (cur === 'SIMD') {
            layer.bindPopup(feature.properties.Name_1).openPopup();
            layer.setStyle({
                color: '#ff0000',
                fillOpacity: 0.1,
                weight: 1,
                opacity: 0.5,
            })

        }
        // Set style of Carbon Storage Layer
        if (cur === 'Carbon') {
            layer.on({
                mouseover: (e) => {
                    let layer = e.target;
                    layer.setStyle({
                        weight: 5,
                        color: 'red',
                        dashArray: '',
                        fillOpacity: 0.7
                    })
                    e.target.openPopup();
                },
                mouseout: (e) => {
                    e.target.closePopup();
                    layer.setStyle({
                        color: 'rgb(198,252,198)',
                        fillOpacity: 1,
                    })
                    if (feature.properties.TonCO2eq > 1000) {
                        layer.setStyle({
                            color: '#47dc47',
                            fillOpacity: 1,
                        })
                    }
                    if (feature.properties.TonCO2eq > 5000) {
                        layer.setStyle({
                            color: '#066206',
                            fillOpacity: 1,
                        })
                    }
                }
            });
            layer.bindPopup("Neighbourhood: " + feature.properties.LABEL + "</br>" + ("Tons of CO&#8322; eq:" + feature.properties.TonCO2eq).slice(0, 29)).openPopup();
            layer.setStyle({
                color: 'rgb(198,252,198)',
                fillOpacity: 1,
            })
            if (feature.properties.TonCO2eq > 1000) {
                layer.setStyle({
                    color: '#47dc47',
                    fillOpacity: 1,
                })
            }
            if (feature.properties.TonCO2eq > 5000) {
                layer.setStyle({
                    color: '#066206',
                    fillOpacity: 1,
                })
            }
        }
        // Set style of All Potential Planting Layer
        if (cur === 'APP') {
            layer.bindPopup(feature.properties.priFunc).openPopup();
            if (feature.properties.Replant_St === 'Remaining') {
                layer.setStyle({
                    color: '#FFA000',
                    fillOpacity: 0,
                    weight: 2,
                })
            }
            if (feature.properties.Replant_St === 'Second Priority') {
                layer.setStyle({
                    color: '#00ffaa',
                    fillOpacity: 0,
                    weight: 2,
                })
            }
            if (feature.properties.Replant_St === 'First Priority') {
                layer.setStyle({
                    color: '#aa00aa',
                    fillOpacity: 0,
                    weight: 2,
                })
            }

        }
    }
    // Set Base Maps
    let osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
    });
    let mapbox = L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw', {
        id: 'mapbox/satellite-v9',
        tileSize: 512,
        zoomOffset: -1,
        attribution: 'Imagery &copy; <a href="https://www.mapbox.com/">Mapbox</a>'
    });
    let google = L.tileLayer('https://{s}.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', {
        subdomains: ['mt0', 'mt1', 'mt2', 'mt3'],
        attribution: 'Imagery &copy; <a href="https://maps.google.com/">Google</a>'
    });
    let view = L.latLng(55.93868, -3.20251);
    // Set Layer Groups
    let SIMD = L.layerGroup();
    let APP = L.layerGroup();
    let Nei = L.layerGroup();
    let Carbon = L.layerGroup();
    // Set Map Frame
    let map = L.map('map', {
        center: view,
        zoom: 12,
        layers: [google, Carbon, SIMD]
    });
    // Load Json Data
    let cur = ''; //Flag of Layer Identification
    jQuery.getJSON('https://cgsdata.annierzhy.workers.dev/Nei.json', function (data) {
        cur = 'Nei';
        L.geoJSON(data, {
                onEachFeature: onEachFeature,
            }
        ).addTo(Nei);
        return true
    });
    jQuery.getJSON('https://cgsdata.annierzhy.workers.dev/Carbon.json', function (data) {
        cur = 'Carbon';
        L.geoJSON(data, {
                onEachFeature: onEachFeature,
            }
        ).addTo(Carbon);
        return true
    });
    jQuery.getJSON('https://cgsdata.annierzhy.workers.dev/APP.json', function (data) {
        cur = 'APP';
        L.geoJSON(data, {
            onEachFeature: onEachFeature,
        }).addTo(APP);
        return true
    });
    jQuery.getJSON('https://cgsdata.annierzhy.workers.dev/SIMD.json', function (data) {
        cur = 'SIMD';
        L.geoJSON(data, {
            onEachFeature: onEachFeature,
        }).addTo(SIMD);
        return true
    });
    //Set Layer Control
    let baseMaps = {
        "OpenStreetMap": osm,
        'MapBox': mapbox,
        'Google': google,
    };
    let overlayMaps = {
        "CarbonStorage": Carbon,
        "SIMD_Zone": SIMD,
        "Neighbourhood": Nei,
        "AllPotentialPlanting": APP,

    };
    let layerControl = L.control.layers(baseMaps, overlayMaps).addTo(map);
    // Set Legends
    let legend = L.control.Legend({
        position: "bottomleft",
        collapsed: false,
        symbolWidth: 24,
        opacity: 1,
        column: 2,
        legends: [{
            label: "Natural Neighbourhood",
            type: "polyline",
            sides: 4,
            color: "#000000",
            weight: 2
        }, {
            label: "SIMD_Zone",
            type: "polyline",
            sides: 4,
            color: "#FF0000",
            fillColor: "#FF0000",
            weight: 2
        }, {
            label: "Remaining Planting Area",
            type: "polygon",
            sides: 4,
            color: "#FFA000",
            fillColor: "#FFA000",
            weight: 2
        }, {
            label: "First Priority Planting Area",
            type: "polygon",
            sides: 4,
            color: "#aa00aa",
            fillColor: "#aa00aa",
            weight: 2
        }, {
            label: "Second Priority Planting Area",
            type: "polygon",
            sides: 4,
            color: "#00ffaa",
            fillColor: "#00ffaa",
            weight: 2
        }, {
            label: "CarbonStorage(0,1000]",
            type: "polygon",
            sides: 4,
            color: "#238E23",
            fillColor: 'rgb(198,252,198)',
            weight: 2
        }, {
            label: "CarbonStorage(1000,5000]",
            type: "polygon",
            sides: 4,
            color: "#238E23",
            fillColor: '#47dc47',
            weight: 2
        }, {
            label: "CarbonStorage>5000",
            type: "polygon",
            sides: 4,
            color: "#238E23",
            fillColor: '#066206',
            weight: 2
        }

        ]
    }).addTo(map);
    // Set Scale Bar
    L.control.scale().addTo(map);
</script>
</html>
```
## Appendix B JavaScripts & CSS
Key Scripts are refered in the section [Detailed Development](#2-detailed-development).
- For Template-related and third-party plug-ins
>See [Source Code](https://www.geos.ed.ac.uk/~s2298227/CGS/assets/) for more details.

## Appendix C Diagram Scripts
### C.1 nei.html
```html
Content-Type: text/html

<!DOCTYPE html>
<html>
<head>
<title>Neighbour</title>
<script src="https://cdn.staticfile.org/echarts/4.3.0/echarts.min.js"></script>
</head>
<body>
<div id="myChart" ref="myChart" style="width: 700px;height:500px;"></div>
</body>
<script>
    let myChart = echarts.init(document.getElementById('myChart'));
    
    let option = {
        title: {
            text: 'Neighbourhood'
        },
        tooltip: {},
        legend: {
            data: ['TonsCO2eq(ESA)','TonsCO2eq(NFI)']
        },
        xAxis: {
            data: [{{nei}}]

        },
        yAxis: {},
        series: [{
            name: 'TonsCO2eq(ESA)',
            type: 'bar',
            data: [{{esa}}]
            },
            {
            name: 'TonsCO2eq(NFI)',
            type: 'bar',
            data: [{{nfi}}]
            }
        ],
        dataZoom: [
            {
                show: true,
                startValue: 0,
                endValue: 20000
            },
            {
                type: 'inside'
            }
        ]
    };

    myChart.setOption(option);
</script>
</html>
```
### C.2 spec.html
```html
Content-Type: text/html

<!DOCTYPE html>
<html>
<head>
<title>Species</title>
<script src="https://cdn.staticfile.org/echarts/4.3.0/echarts.min.js"></script>
</head>
<body>
<div id="myChart" ref="myChart" style="width: 700px;height:500px;"></div>
</body>
<script>
    let myChart = echarts.init(document.getElementById('myChart'));
    
    let option = {
        title: {
            text: 'Specie'
        },
        tooltip: {},
        legend: {
            data: ['TonsCO2eq']
        },
        xAxis: {
            data: [{{spec}}]

        },
        yAxis: {},
        series: [{
            name: 'TonsCO2eq',
            type: 'bar',
            data: {{num}}
            },
        ],
        dataZoom: [
            {
                show: true,
                startValue: 0,
                endValue: 20000
            },
            {
                type: 'inside'
            }
        ]
    };

    myChart.setOption(option);
</script>
</html>
```
### C.3 nei.py
```python
#!/usr/bin/env python3
import cgitb
import cx_Oracle
cgitb.enable()
from jinja2 import Environment, FileSystemLoader

def print_html():
    env = Environment(loader=FileSystemLoader('.'))
    temp = env.get_template('nei.html')
    a,b,c = carbonHtml()
    print(temp.render(nei = a,nfi = b, esa = c))

def carbonHtml():
    with open("oracle",'r') as pwf:
        db = pwf.readline().strip()
        usr = pwf.readline().strip()
        pw = pwf.readline().strip()
    conn = cx_Oracle.connect(dsn = db,user = usr,password = pw)
    c = conn.cursor()
    c.execute("SELECT * FROM CARBON_NEI")
    data = ''
    n = ''
    e = ''
    index = ''
    
    for row in c:
        if index != row[1]:
            data = data + '"' + row[1] + '"' + ','
        index = row[1]
        if row[3] == 'N\r':
                n = n + str(row[2]) +','
        if row[3] == 'E\r':
                e = e + str(row[2]) +','
                
    conn.close()
    return data,n,e

if __name__ == '__main__':
    print_html()
```
### C.4 spec.py
```python
#!/usr/bin/env python3
import cgitb
import cx_Oracle
cgitb.enable()
from jinja2 import Environment, FileSystemLoader

def print_html():
    env = Environment(loader=FileSystemLoader('.'))
    temp = env.get_template('spec.html')
    a,b = carbonHtml()
    print(temp.render(spec = a,num = b))

def carbonHtml():
    with open("oracle",'r') as pwf:
        db = pwf.readline().strip()
        usr = pwf.readline().strip()
        pw = pwf.readline().strip()
    conn = cx_Oracle.connect(dsn = db,user = usr,password = pw)
    c = conn.cursor()
    c.execute("SELECT * FROM SPEC")
    data = ''
    calc = 0
    num = []
    index = ''
    
    for row in c:
        if index != row[1]:        
            data = data + '"' + row[1] + '"' + ','
            num.append(calc)
            calc = 0
        index = row[1]
        if index == row[1]:
            calc = calc + row[3] 
    num.append(calc)
    conn.close()
    return data,num[1:]

if __name__ == '__main__':
    print_html()
```
## Appendix D Database Scripts
### D.1 NEI.sql
```sql
CREATE TABLE CARBON_NEI(
ID NUMBER(3) NOT NULL,
NEIHOOD VARCHAR2(40) NOT NULL,
TOTALC NUMBER(20,5),
SOURCE VARCHAR2(4),
PRIMARY KEY(ID)
)
```
### D.2 SPEC.sql
```sql
CREATE TABLE SPEC(
ID NUMBER(3) NOT NULL,
TYPENAME VARCHAR2(40) NOT NULL,
NEI VARCHAR2(35),
TOTALC NUMBER(20,5),
PRIMARY KEY(ID)
)
```
### D.3 NEI_LDR.ctl
```sql
OPTIONS(skip=1)
LOAD DATA
INFILE 'NEI.csv'
REPLACE
INTO TABLE CARBON_NEI
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(ID,NEIHOOD,TOTALC,SOURCE)
```
### D.4 SPEC_LDR.CTL
```sql
OPTIONS(skip=1)
LOAD DATA
INFILE 'SPEC.csv'
REPLACE
INTO TABLE SPEC
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(ID,TYPENAME,TOTALC,NEI)
```
## Appendix D Manual Pages
>See [Source Code](https://www.geos.ed.ac.uk/~s2298227/CGS/manual/) for more details.


