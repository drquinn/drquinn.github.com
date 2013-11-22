---
layout: post
title: How to Build Single Page Application (SPA) in SharePoint Using Durandal
---

Single Page Apps (SPA) are becoming many SharePoint and .NET developers [favorite](http://www.andrewconnell.com/blog/sharepoint-hosted-apps-in-sp-2013-as-single-page-apps) way to [build](http://www.johnpapa.net/spa/) applications.  There are many great frameworks to aid in the process, but one of my favorites is [Durandal](http://durandaljs.com/).  Durndal itself relies on some other very well known libraries, such as [Knockout](http://knockoutjs.com/), [Require.js](http://requirejs.org/), and [jQuery](http://jquery.com/).  Here I will show how to create a Durandal based SPA that can be deployed into SharePoint as a [SharePoint-hosted app](http://msdn.microsoft.com/en-us/library/fp142379.aspx).  This means all code will live client side and we will not need any access to the server or server side code.


###Building and Understanding SharePoint-hosted apps

Open Visual Studio 2013 and create a new project.  Create a C# project with the .NET Framework 4.5.  I will be creating an expense report manager to demonstrate the capibilities of a SPA in SharePoint, so I named my project ReportManager.

![New SharePoint Project](/assets/HowtoDurandalSP/newproject.png "New SharePoint Project")

Next, you need to point your app to a SharePoint site.  I'll be deploying this app to a site collection called "Finance."


![New SharePoint App](/assets/HowtoDurandalSP/newapp.png "New SharePoint App").


The great thing about Visual Studio 2013 templates is that they have enough code to be deployed immediately and function without any additional work from you.  Therefore, I always deploy the app right after creating to ensure my environment is working.  To do this, right click your project and click "Deploy" or just press F5 to deploy your app in debug mode.  I'll be using F5 to deploy.

You should see a screen like this after deploying.  My username is "SharePoint Setup," so we can see the template uses the client side SharePoint [Javascript Object Model](http://msdn.microsoft.com/en-us/library/office/jj163201.aspx#BasicOps_SPJSOMOps) to get the current user and display it after Hello.

![Start Screen](/assets/HowtoDurandalSP/start1.png "Start Screen").

We now have a working SharePoint-hosted app.  Before we move forward lets explore the template Microsoft has created for us.  Open the AppManifest.xml file.  This keeps some general information about the app like the title, version, and icon path. But more importantly, it points to our Start page and query string.  The start page is set to default.aspx and located in the Pages folder.

![App Manifest](/assets/HowtoDurandalSP/appmanifest.png "App Manifest").

The Query string parameter is also very important to us.  It is set to {StandardTokens} by defualt and that will work just fine for us.  SharePoint has about 30 tokens that can be placed at the beginning or middle of URLs to automatically pull information from the SharePoint farm.  For example, ~site is a token that gets the URL of the current website and ~layouts gets the Layouts virtual folder.

{StandardTokens} combines 5 other tokens into one and passes them into your URL for us to use later: 
	
	SPHostUrl={HostUrl}&SPAppWebUrl={AppWebUrl}&SPLanguage={Language}&SPClientTag={ClientTag}&SPProductNumber={ProductNumber}

Back to default.aspx, we see a variety of JavaScript files being called, such as jQuery by defalut.  A content place holder for the page title and for the main body.  In the body, we only see a paragraph tag and text that says 'initializing...'  The comment explains that the content is replace on load and to see App.js.  This is where SharePoint is storing the custom logic for the application.


So how do we transform this into something like Gmail, Outlook.com, or any of the SPAs you might imagine?  We need to create views, navigation, and read/write data.  That's where Durandal comes in.  Let's get it with Visual Studio's package manager [NuGet](http://www.nuget.org/).  NuGet provides a simple way to retrieve these JavaScript and CSS libraries, automatically add them to our project, and keep them updated.  Before using NuGet, take a look at the Scripts folder included by the SharePoint app template.  Make a note of the JavaScript files included and watch how they update next.

Open the NuGet Package Manager Console under Tools > Library Package Manager > Package Manager Console.

![NuGet](/assets/HowtoDurandalSP/nuget.png "NuGet").

To find out which package we need to download, lets get a list of all packages that use Durandal by typing:
	
	Get-Package Durandal -ListAvailable

We see Durandal in the list, currently at version 2.0.1.  To install it, type:
	
	Install-Package Durandal.StarterKit

Notice that Durandal depends on other libraries, like knockoutjs and jQuery.  NuGet checks our project and because they are not installed, it automatically installs them for us.  At this point, it is good to point out that we could have just as easily gone to each of those libraries sites, downloaded the required js files, and added them to our scripts folder manually.  But wasn't this much easier?  Check the scripts folder to see the new JavaScript files downloaded.  Durandal will now handle all of the plumbing for the application, doing things like managing views, navigation, loading JavaScript modules only when needed, and the applicaiton life cycle. (i.e. what happens when a page first loads or when a user tries to navigate away)

Also install - Toastr:

	Install-Package Toastr

The starter kit installed a few other things for us, including Bootstrap and Font Awesome which help us get a great looking user interface running in seconds.  Hit F5.  Ah, breaks:

![No MVC](/assets/HowtoDurandalSP/nomvc.png "No MVC").

We won't be doing anything server side in this project, so we are not using controllers in that way.  We will be getting our data from SharePoint lists using REST, as we will see later.  So just delete the Controllers folder.  Hit F5 again.  The page comes up, but it is not using Durandal.  That's because the starter kit assumes we will use Index.cshtml under Views/Durandal/Index.cshtml.  But we are not, we are sticking to the load file that SharePoint provides at Pages/Default.aspx.

Now add the references to all those CSS and JS files we just added:

![JS CSS](/assets/HowtoDurandalSP/jscss.png "JS CSS").

I leave the default JS and CSS files, except App.css and App.js which we will not be needing any longer.  

Next, we point require.js to our main starting file, and that file will load up Durandal and any modules we define.  Add this script tag to the bottom of the body:

	<script type="text/javascript" src="../Scripts/require.js" data-main="../App/main"></scirpt>


This will tell Durandal to load.  After it loads, it will render all content into a div with the id applicationHost.  So add that div as well above the require.js script tag:

	<div id="applicationHost">
		<div>
			Page loading...
		</div>
	</div>

And that quickly, Durandal should be up and functional.  Run the app.  You should have a grey navigation bar across the top.  Click Flickr to see the picture viewer show.  You may notice that the SharePoint navigation top bar is blocked.  To unblock that, open shell.html and located the second div tag from the top.  It has the class navbar-fixed-top applied to it.  Delete that class and rerun the app.  The nav should now have dropped down below the title.

###Hook up with SharePoint Data

Next, lets have the App create lists and then show data from those lists using REST.
Right click the project and click Add New Item.  Choose List and name it reports.  Create a template and instance with the default option "Custom List".

![SharePoint List](/assets/HowtoDurandalSP/list.png "SharePoint List").

This will automatically take you into the template designer.  We have title, lets add an additional column called "Requestor" and keep it "Single Line of Text" for simplicity.  Now add data by opening up the Elements.xml file under reportsInstance.  Update it to look like this:

	<?xml version="1.0" encoding="utf-8"?>
	<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
	  <ListInstance Title="reports" OnQuickLaunch="TRUE" TemplateType="10000" Url="Lists/reports" Description="My List Instance">
	    <Data>
	      <Rows>
	        <Row>
	          <Field Name="Title">San Francisco Trip</Field>
	          <Field Name="Requestor">Brent Long</Field>
	          <Field Name="Cost">890</Field>
	        </Row>
	        <Row>
	          <Field Name="Title">Client Lunch</Field>
	          <Field Name="Requestor">Bert Solano</Field>
	          <Field Name="Cost">65</Field>
	        </Row>
	        <Row>
	          <Field Name="Title">SharePoint Conference</Field>
	          <Field Name="Requestor">Christian Kilyk</Field>
	          <Field Name="Cost">2300</Field>
	        </Row>
	      </Rows>
	    </Data>
	  </ListInstance>
	</Elements>

Press F5 and take a look at the application.  You should be able to navigate to the list.  But notice that the list is not available on the root site that we deployed to.  The list is actually deployed to the app web that is created for us.


![SharePoint List Active](/assets/HowtoDurandalSP/listactive.png "SharePoint List Active")

### Set Up the Initial View

Now we need to connect to the list we just created.  We do this by creating a new view.  Expand the App folder off your project root and right click on the "view" folder, choose Create New Item.  Create an HTML Page and name it "reports".  We don't need any of the template text so delete it and add the follow text.
	
	<section class="view">
	    <header>
	        <h2 data-bind="text: title"></h2>
	        <span data-bind="text: reports().length"></span><span> Reports</span>
	        <br /><br />
	        <button type="button" class="btn btn-primary" data-bind="click: create">
	            <i class="icon-plus"></i>
	        </button>
	        <br /><br />
	    </header>
	    <section>
	        <div class="table-responsive">
	            <table class="table table-striped table-hover">
	                <tbody data-bind="foreach: reports">
	                    <tr>
	                        <td><a data-bind="attr: { href: '#/item/' + Id() }"><span data-bind="text: Title" /></a></td>
	                        <td><span data-bind="text: Requestor" /></td>
	                        <td><span data-bind="text: Cost" /></td>
	                    </tr>
	                </tbody>
	            </table>
	        </div>
	    </section>
	</section>


### Set Up the Initial View Model

Add the JS code needed for the View Model.  This includes activate and deactivate methods and a call to get the reports using jQuery and AJAX.

	define(['durandal/app', 'durandal/system', 'plugins/router'],
	    function (app, system, router) {
	        var reports = ko.observableArray([]);
	        var Report = function (dto) {
	            // Map to obesrvables and add computed observables
	            return addReportsComputed(
	                mapToObservable(dto));
	        };
	
	        // ### A) Add Create Method, B) Map to VM, C, Add route, D) Add router reference ###
	        var create = function () {
	            router.navigate('report-details');
	        };
	
	        var activate = function () {
	            if (reports().length > 0) {
	                toastr.success('Already Loaded');
	                return;
	            };
	
	            return getReports();
	        };
	
	        var getReports = function () {
	            // 1. ### Set Ajax options ###
	            var options = {
	                url: "../_api/lists/getByTitle('reports')/Items",
	                type: 'GET',
	                dataType: 'json',
	                headers: { "Accept": "application/json; odata=verbose" }
	            };
	
	            // 2. ### Make call ###
	            return $.ajax(options)
	                        .then(querySucceeded)
	                        .fail(queryFailed);
	
	            // 3. ### Handle response ###
	            function querySucceeded(data) {
	                var reportsArray = [];
	                var results = data.d.results;
	                results.forEach(function (item) {
	                    var e = new Report(item);
	                    reportsArray.push(e);
	                });
	                reports(reportsArray);
	                system.log('Retrieved report observables');
	                toastr.success("Reports Refreshed");
	            };
	        };
	
	        function mapToObservable(dto) {
	            var mapped = {};
	            for (prop in dto) {
	                if (dto.hasOwnProperty(prop)) {
	                    mapped[prop] = ko.observable(dto[prop]);
	                }
	            }
	            return mapped;
	        };
	
	        function addReportsComputed(entity) {
	            entity.adjustedCreated = ko.computed(function () {
	                return entity.Created().split('T')[0];
	            });
	            return entity;
	        };
	
	        function queryFailed(jqXHR, textStatus) {
	            var msg = 'Error retrieving data.' + textStatus;
	            toastr.error(msg);
	        };
	
	        // ### Setup for deactivation ###
	
	        var vm = {
	            activate: activate,
	            reports: reports,
	            title: "Reports Page",
	            create: create
	        };
	
	        return vm;
	    });

### Set up the Navigation

Now we need to navigate to our new view.  This is done using a part of Durandal called the router.  We simply need to add our route and tell the router to include the route in our default navigation.

Open **shell.js** and update router.map to look like this:

	router.map([
                { route: '', title:'Welcome', moduleId: 'viewmodels/welcome', nav: true },
                { route: 'flickr', moduleId: 'viewmodels/flickr', nav: true },
                { route: 'reports', moduleId: 'viewmodels/reports', nav: true }
            ]).buildNavigationModel();


###Set up Create, Update, Delete

We just figured out how to read data.  Now we will finsih the CRUD set.  Create a new html page in the views folder called report-details.html and a new JavaScript page called **report-details.js**.  Add the follow code to each page:



###Set up Update

Add the following code to **report-details.js**:
	
	define(['durandal/app', 'durandal/system', 'plugins/router', 'viewmodels/reports'],
	    function (app, system, router, reportsVM) {
	
	        var ReportInitialize = {
	            Title: "",
	            Requestor: "",
	            Cost: "",
	            Created: ""
	        };
	        var Report = function (dto) {
	            // Map to obesrvables and add computed observables
	            return addReportsComputed(
	                mapToObservable(dto));
	        };
	        var details = ko.observableArray([ReportInitialize]);
	        var newItem = ko.observable(false);
	
	        // Activate called when report-details is loaded
	        var activate = function (Id) {
	
	            if (!Id) {
	                newItem(true);
	                clearDetails();
	                return;
	            }
	
	            return filterReports(Id);
	        };
	
	        var create = function () {
	            $$.ajax({
	                url: "../_api/lists/getByTitle('reports')/Items",
	                type: "POST",
	                data: JSON.stringify({
	                    '__metadata': { 'type': 'SP.Data.ReportsListItem' },
	                    'Title': details()[0].Title,
	                    'Requestor': details()[0].Requestor,
	                    'Cost': details()[0].Cost
	                }),
	                headers: {
	                    'accept': 'application/json;odata=verbose',
	                    'content-type': 'application/json;odata=verbose',
	                    'X-RequestDigest': $$('#__REQUESTDIGEST').val()
	                },
	                success: function (data) {
	                    toastr.success("New Report Added");
	                    reportsVM.reports([]);
	                    clearDetails();
	                    router.navigate('reports');
	                },
	                error: function (err) {
	                    alert(JSON.stringify(err));
	                }
	            });
	        };
	
	        var save = function () {
	
	            var metadata = {
	                'Title': details()[0].Title(),
	                'Requestor': details()[0].Requestor(),
	                'Requestor': details()[0].Cost()
	            };
	            var item = $$.extend({
	                "__metadata": { 'type': 'SP.Data.ReportsListItem' }
	            }, metadata);
	
	            $$.ajax({
	                url: details()[0].__metadata().uri,
	                type: "POST",
	                contentType: "application/json;odata=verbose",
	                data: JSON.stringify(item),
	                headers: {
	                    'X-HTTP-Method': 'MERGE',
	                    'accept': 'application/json;odata=verbose',
	                    'X-RequestDigest': $$("#__REQUESTDIGEST").val(),
	                    'IF-MATCH': "*"
	                },
	                success: function () {
	                    toastr.success('Report Saved');
	                    router.navigate('reports');
	                },
	                error: function (err) {
	                    toastr.error(JSON.stringify(err));
	                }
	            });
	        };
	
	        var filterReports = function (Id) {
	            details(ko.utils.arrayFilter(reportsVM.reports(), function (item) {
	                return item.Id() == Id;
	            }));
	            toastr.success('Reports Filtered');
	        };
	
	        var deleteReport = function () {
	            $$.ajax({
	                url: details()[0].__metadata().uri,
	                type: "POST",
	                headers: {
	                    'X-HTTP-Method': 'DELETE',
	                    'accept': 'application/json;odata=verbose',
	                    'content-type': 'application/json;odata=verbose',
	                    'X-RequestDigest': $$('#__REQUESTDIGEST').val(),
	                    'IF-MATCH': details()[0].__metadata().etag
	                },
	                success: function () {
	                    toastr.success("Report Deleted Successfully");
	                    reportsVM.reports([]);
	                    clearDetails();
	                    router.navigate('reports');
	                },
	                error: function (err) {
	                    toastr.error(JSON.stringify(err));
	                }
	            });
	        };
	
	        function clearDetails() {
	            details([]);
	            details.push({
	                Title: "",
	                Requestor: "",
	                Cost: "",
	                Created: ""
	            });
	        };
	
	        function mapToObservable(dto) {
	            var mapped = {};
	            for (prop in dto) {
	                if (dto.hasOwnProperty(prop)) {
	                    mapped[prop] = ko.observable(dto[prop]);
	                }
	            }
	            return mapped;
	        };
	
	        function addReportsComputed(entity) {
	            entity.adjustedCreated = ko.computed(function () {
	                return entity.Created().split('T')[0];
	            });
	            return entity;
	        };
	
	        function queryFailed(jqXHR, textStatus) {
	            var msg = 'Error retrieving data.' + textStatus;
	            toastr.error(msg);
	        };
	
	        var canDeactivate = function () {
	            return app.showMessage('Are you sure you want to leave this page?', 'Navigate', ['Yes', 'No']);
	        };
	
	        var detailsVM = {
	            activate: activate,
	            canDeactivate: canDeactivate,
	            title: "Report Details",
	            details: details,
	            create: create,
	            save: save,
	            deleteReport: deleteReport,
	            newItem: newItem
	        };
	
	        return detailsVM;
	    });

Update the **reports.html** view with a link that will pass the id:
	
	<a data-bind="attr: { href: '#/item/' + Id() }">

Update the **reports-detail.html** view with the new buttons we have created:

	<div>
	    <h4>Report Details</h4>
	    <div data-bind="foreach: details">
	        <div class="form-group">
	            <label for="expenseTitle">Report Title</label>
	            <input class="form-control" type="text" data-bind="value: Title" id="expenseTitle" placeholder="Title" />
	        </div>
	        <div class="form-group">
	            <label for="expenseRequestor">Requestor</label>
	            <input class="form-control" type="text" data-bind="value: Requestor" id="expenseRequestor" placeholder="Requestor" />
	        </div>
	        <div class="form-group">
	            <label for="expenseCost">Cost</label>
	            <input class="form-control" type="text" data-bind="value: Cost" id="expenseCost" placeholder="Cost" />
	        </div>
	        <!-- ko if: Created -->
	        <div class="form-group">
	            <label for="expenseCreated">Report Created</label>
	            <input class="form-control" type="text" data-bind="value: Created" id="expenseCreated" placeholder="Created" readonly="readonly" />
	        </div>
	        <!-- /ko -->
	    </div>
	    <br /><br />
	    <!-- ko if: newItem() == true -->
	    <button type="button" class="btn btn-primary" data-bind="click: create">
	        Create
	    </button>
	    <!-- /ko -->
	    <!-- ko if: newItem() == false -->
	    <button type="button" class="btn btn-primary" data-bind="click: save">
	        <i class="icon-save"></i>
	    </button>
	    &nbsp;&nbsp;&nbsp;
	    <button type="button" class="btn btn-primary" data-bind="click: deleteReport">
	        <i class="icon-remove"></i>
	    </button>
	    <!-- /ko -->
	    <br /><br />
	</div>
	
	
And lastly, update your shell.js file with the new routes:

	router.map([
	                { route: '', title:'Welcome', moduleId: 'viewmodels/welcome', nav: true },
	                { route: 'flickr', moduleId: 'viewmodels/flickr', nav: true },
	                { route: 'reports', moduleId: 'viewmodels/reports', nav: true },
	                { route: 'report-details', moduleId: 'viewmodels/report-details', nav: false },
	                { route: 'item/:Id', moduleId: 'viewmodels/report-details', nav: false }
	            ]).buildNavigationModel();
	           
	           
And with that complete, you now have a working CRUD SPA using Durandal running as a SharePoint 2013 Hosted App.

