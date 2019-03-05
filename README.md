# Live Project

## Introduction

For the final project at The Tech Academy, I was assigned along with a group of my peers to work on a full-scale MVC/MVVM project building out an interface for Job Placement at The Tech Academy.
The project would help automate application tracking and hires in the job placement course both to enhance user experience for future students as well as to help pull meaningful data to provide
students with relevant analytics on job placement stats.  It was a great opportunity to work with a legacy code base, see how the code was laid out, fix any bugs, and add new features. 
I also got to see how a project can change over time and outgrow some of choices made in earlier stages of the project.  We worked together as a team, sharing any ideas or struggles we 
encountered on parts of the projects we were each working on. During the two weeks on the project I worked on several stories both front end and back end.  Because we had a decent sized group and limited time,
we shared stories in order to give everyone an opportunity to try working on the front end and back end.  Over the two weeks I also had the opportunity to work on other project and team programming skills that I'm
confident I'll use again on future projects.

Below are descriptions of stories I helped work on, along with code snippets and navigation links.  

## Back End Stories
* [Add Display Names](#add-display-names)
* [Email Students Button Updates Database](#email-students-button-updates-database)
* [Refactored Code](#refactored-code)

## Front End Stories
* [Align Name Column](#align-name-column)
* [Move Email Button](#move-email-button)
* [Added header to JPBulletins create view](#added-header-to-jpbulletins-create-view)
* [Added Back to List Button](#added-back-to-list-button)
* [Added Margins to View](#added-margins-to-view)
* [Removed link from Index](#removed-link-from-index)
* [Edit Student Drop Down](#edit-student-drop-down)


# Back End Stories

### Add Display Names
First back end story I took a stab at was adding display names to the JPCheclist.cs


{
        public int JPChecklistid { get; set; }
        public string ApplicationUserid { get; set; }
        [DisplayName("Business Cards")]
        public bool JPBusinessCards { get; set; }
        [DisplayName("Meetups")]
        public bool JPMeetups { get; set; }
        [DisplayName("Updated LinkedIn")]
        public bool JPUpdatedLinkedIn { get; set; }
        [DisplayName("Updated Portfolio Site")]
        public bool JPUpdatedPortfolioSite { get; set; }
        [DisplayName("Clean GitHub")]
        public bool JPCleanGitHub { get; set; }
        [DisplayName("Round Tables")]
        public bool JpRoundTables { get; set; }

### Email Students Button Updates Database
(dB.LatestContact)

JPNotificationsController.cs 

// open email link
            return Redirect(email);
        }
       -- public List<string> listOfEmails(string emailList) --
        
     **   public ActionResult EmailAll (string userid, string emailList) **
        {
            List<string> listOfEmails = emailList.Split(',').ToList();
            foreach (var emailName in listOfEmails)
            {
                var userId = db.JPStudents.Where(x => x.JPEmail == emailName).First().ApplicationUserId.ToString();
                UpdateLatestContact(userId);
            }
            return listOfEmails;
            EmailHelper.listOfEmails (emailList);
            EmailHelper.UpdateLatestContact(userid);
            EmailHelper.SendMail(emailList);
            EmailHelper.UpdateAndBcc(emailList);
            return RedirectToAction("Index");
        }
       
        private void UpdateLatestContact(string userId)
        {
            throw new NotImplementedException();
        }
        protected override void Dispose(bool disposing)
        {

Helper.cs

			
    public static class EmailHelper
    {
        private static ApplicationDbContext db = new ApplicationDbContext();
        public static List<string> listOfEmails(string emailList)
        {
            List<string> listOfEmails = emailList.Split(',').ToList();
            foreach (var emailName in listOfEmails)
            {
                var userId = db.JPStudents.Where(x => x.JPEmail == emailName).First().ApplicationUserId.ToString();
                UpdateLatestContact(userId);
            }
            return listOfEmails;
        }
        public static void UpdateLatestContact(string userid)
        {
            //update latest contact if entry found
            if (db.JPLatestContacts.Where(c => c.ApplicationUserId == userid).ToList().Count() > 0)
            {
                var contact = db.JPLatestContacts.Where(c => c.ApplicationUserId == userid).FirstOrDefault();
                contact.JPLatestContactDate = DateTime.Now;
                db.SaveChanges();
            }
            //otherwise add new latestContactObject
            else
            {
                var contact = new JPLatestContact();
                contact.ApplicationUserId = userid;
                contact.JPLatestContactDate = DateTime.Now;
                db.JPLatestContacts.Add(contact);
                db.SaveChanges();
            }
        }
        public static List<string> SendMail(string emailList)
        {
            List<string> listOfEmails = emailList.Split(',').ToList();
            foreach (var emailName in listOfEmails)
            {
                //Takes each email in list and searches for it on the JPStudents table and finds the associated ApplicationUserID.
                //Then calls the UpdateLatestContact method on each ApplicationUserID.
                var userId = db.JPStudents.Where(x => x.JPEmail == emailName).First().ApplicationUserId.ToString();
                UpdateLatestContact(userId);
            }
            return listOfEmails;
        }
        [HttpPost]
        public static void UpdateAndBcc(string emailList)
        {
            //Calls the SendMail function above to ensure latest contact info is used then sends the updated list to email app as BCC.
            var mailingList = SendMail(emailList);
            string mailString = "mailto:?bcc=";
            string[] compileEmail = mailingList.ToArray();
            mailString += String.Join(",", compileEmail); //Using an array plus String.Join avoids a trailing comma.
            System.Diagnostics.Process.Start(mailString);
            return;
            //In the future if this call is made via Ajax and no return is needed, just delete return and change type to void.
        }
    }
    public static class DateCalculateHelper
    {
        public static bool? CalculateIsObjectCreatedWithinOneWeekOfCurrentDate(DateTime? objectCreatedDate)

Updates.cshtml 

			
                        </td>
                    </tr>
                }
                @{
                    int strLength = emailList.Length;
                    emailList = emailList.Substring(0, (strLength - 1)); //removes last comma
                }
            </table>
        <div>
            --    <form style="display: inline" action="mailto:?bcc=@emailList" method="post">
           
        **    <form style="display: inline" action="@Url.Action("EmailAll", new { emailList })" method="post">
                <button class="btn btn-primary">Email Students</button>
            </form>
        </div>
    </div>
</div>
@section Scripts {
    
    <script type="text/javascript">
        $(".dropdown-toggle").dropdown();
    </script>

### Refactored Code

JPStudentController.cs 

JPStudentRundownController.cs ------ BEFORE

public ViewResult Index(string sortOrder, string searchString, string latestContact)
        {
            ViewBag.NameSortParm = String.IsNullOrEmpty(sortOrder) ? "name_desc" : "";
            ViewBag.DateSortParm = sortOrder == "Date" ? "date_desc" : "Date";
            ViewBag.LocationSortParm = sortOrder == "Location" ? "location_desc" : "Location";
            ViewBag.GraduateSortParm = sortOrder == "Ready To Graduate" ? "graduate_desc" : "Graduate";
            ViewBag.TotalSortParm = sortOrder == "Total" ? "total_desc" : "Total";
            ViewBag.WeeklySortParm = sortOrder == "Weekly" ? "weekly_desc" : "Weekly";
            ViewBag.NoActivity = sortOrder == "No Activity" ? "No_Activity" : "No_Activity";
            ViewBag.LatestContactSortParm = sortOrder == "LatestContact" ? "latestcontact_desc" : "LatestContact";
            List<JPStudentRundown> jPStudentRundownList = new List<JPStudentRundown>();
            var students = from s in db.JPStudents
                           where s.JPHired == false //setting the logic here so we can skip the whole passing of the object into the list, and then not have to look at it in the view at all. Before it would still have to hit all the information below
                           select s;                        //where as now it will skip the following and not even be there in the view
            {
                int jpid = Convert.ToInt32(latestContact);
                string userid = db.JPStudents.Where(s => s.JPStudentId == jpid).FirstOrDefault().ApplicationUserId;
                UpdateLatestContact(userid);
            }
            switch (sortOrder)
            {
                case "Graduate":
                default: //Name ascending
                    students = students.OrderBy(s => s.JPName);
                    break;
            }
            foreach (var student in students)
            {
                if (sortOrder == "Graduate" || sortOrder == "graduate_desc")
                    jPStudentRundownList.Add(BuildRundownObj(student.JPStudentId));
                }
            }
            return View(jPStudentRundownList);
        }
        // GET: JPStudentRundown/Edit/5
            return studentRundown;
        }
        public void UpdateLatestContact(string userid)
        {
            EmailHelper.UpdateLatestContact(userid);
            return;
        }
        public List<string> SendMail(string emailList)
        {
            return EmailHelper.SendMail(emailList);
        }
        [HttpPost]
        public ActionResult UpdateAndBcc(string emailList, string body)
        {
            //Calls the SendMail function above to ensure latest contact info is used then sends the updated list to email app as BCC.
            var mailingList = SendMail(emailList);
            string mailString = "mailto:?bcc=";
            string[] compileEmail = mailingList.ToArray();
            mailString += String.Join(",", compileEmail); //Using an array plus String.Join avoids a trailing comma.
            mailString += body;
            System.Diagnostics.Process.Start(mailString);
            return RedirectToAction("Index");
            //In the future if this call is made via Ajax and no return is needed, just delete return and change type to void.
        }
        //Counts the total of completed items in JPChecklists table




JPStudentRundownController.cs --- AFTER

		
        public ViewResult Index(string sortOrder, string searchString, string latestContact)
        {
            var students = from s in db.JPStudents
                           where s.JPHired == false //setting the logic here so we can skip the whole passing of the object into the list, and then not have to look at it in the view at all. Before it would still have to hit all the information below
                           select s;                        //where as now it will skip the following and not even be there in the view
            {
                int jpid = Convert.ToInt32(latestContact);
                string userid = db.JPStudents.Where(s => s.JPStudentId == jpid).FirstOrDefault().ApplicationUserId;
                EmailHelper.UpdateLatestContact(userid);
            }
            students = StudentSort(students, sortOrder);
            List<JPStudentRundown> jPStudentRundownList = StudentRundownPopulate(students, sortOrder);
            return View(jPStudentRundownList);
        }
        private IQueryable<JPStudent> StudentSort(IQueryable<JPStudent> students, string sortOrder)
        {
            ViewBag.NameSortParm = String.IsNullOrEmpty(sortOrder) ? "name_desc" : "";
            ViewBag.DateSortParm = sortOrder == "Date" ? "date_desc" : "Date";
            ViewBag.LocationSortParm = sortOrder == "Location" ? "location_desc" : "Location";
            ViewBag.GraduateSortParm = sortOrder == "Ready To Graduate" ? "graduate_desc" : "Graduate";
            ViewBag.TotalSortParm = sortOrder == "Total" ? "total_desc" : "Total";
            ViewBag.WeeklySortParm = sortOrder == "Weekly" ? "weekly_desc" : "Weekly";
            ViewBag.NoActivity = sortOrder == "No Activity" ? "No_Activity" : "No_Activity";
            ViewBag.LatestContactSortParm = sortOrder == "LatestContact" ? "latestcontact_desc" : "LatestContact";
            switch (sortOrder)
            {
                case "Graduate":
                default: //Name ascending
                    students = students.OrderBy(s => s.JPName);
                    break;
            }
            return (students);
        }
        private List<JPStudentRundown> StudentRundownPopulate(IQueryable<JPStudent> students, string sortOrder)
        {
            List<JPStudentRundown> jPStudentRundownList = new List<JPStudentRundown>();
            foreach (var student in students)
            {
                if (sortOrder == "Graduate" || sortOrder == "graduate_desc")
                    jPStudentRundownList.Add(BuildRundownObj(student.JPStudentId));
                }
            }
            return (jPStudentRundownList);
        }
        // GET: JPStudentRundown/Edit/5
            return studentRundown;
        }
        public ActionResult EmailAll(string userid, string emailList)
        {
            EmailHelper.listOfEmails(emailList);
            EmailHelper.UpdateLatestContact(userid);
            EmailHelper.SendMail(emailList);
            EmailHelper.UpdateAndBcc(emailList);
            return RedirectToAction("Index");
        }


Index.cshtml --- BEFORE 

 $(".dropdown-toggle").dropdown();
    </script>
    <!-- JS to toggle radio options when email button clicked-->
    <script type="text/javascript">
        $("#email-students-popup").hide();
            radioInput = $("input[name='email']:checked").val();
            if (radioInput == "graduating") {
                    $("#submit-form").attr("action", "@Html.Raw(@Url.Action("UpdateAndBcc", "JPStudentRundown", new { emailList = emailList , body = gradBody }))" );
                    document.getElementById("submit-email").submit();
                }
                else if (radioInput == "applying") {
                    $("#submit-form").attr("action", "@Html.Raw(@Url.Action("UpdateAndBcc", "JPStudentRundown", new { emailList =emailList, body = applyBody }))" );
                    document.getElementById("submit-email").submit();
                }
                else {


Index.cshtml -- AFTER

 $(".dropdown-toggle").dropdown();
    </script>
    <!-- JS to toggle radio options when email button clicked-->
    <script type="text/javascript">
        $("#email-students-popup").hide();
            radioInput = $("input[name='email']:checked").val();
            if (radioInput == "graduating") {
                    $("#submit-form").attr("action", "@Html.Raw(@Url.Action("EmailAll", "JPStudentRundown", new { emailList = emailList , body = gradBody }))" );
                    document.getElementById("submit-email").submit();
                }
                else if (radioInput == "applying") {
                    $("#submit-form").attr("action", "@Html.Raw(@Url.Action("EmailAll", "JPStudentRundown", new { emailList =emailList, body = applyBody }))" );
                    document.getElementById("submit-email").submit();
                }
                else {


////


# FRONT END STORIES

### Align Name Column
One of the first front end stories I worked on was a small edit, aligning the name column from center to the leftside of the table. 

	// Before
			
                    <td>
                        <i class="@iStyle" style="font-size:14px;color:darkorange"></i>
                    </td>
                    <td class="text-center">
                        @Html.DisplayFor(modelItem => item.StudentName)
                    </td>
               		
               		

	//After
			<td>
                        <i class="@iStyle" style="font-size:14px;color:darkorange"></i>
                    </td>
                    <td class="text-left">
                        @Html.DisplayFor(modelItem => item.StudentName)
                    </td>




### Move Email Button
Another front end story I worked on was moving a button that would export student email addresses below the 'Weekly Hires' table.

 Move 'export student email addresses' button below Weekly Hires table. 

       </tr>
            }
        </table>
        @Html.ActionLink("Export Student Email Addresses", "ExportCSV", null, new { @class = "btn btn-primary" })
    </div>
</div>
-------------------------------------------------------------------------------------------------------------
                </tr>
            }
        </table>
        @Html.ActionLink("Export Student Email Addresses", "ExportCSV", null, new { @class = "btn btn-primary" })
        
    </div>
</div>

### Added header to JPBulletins create view

 ////////

@model JobPlacementDashboard.Models.JPBulletin
<h3>&nbsp;&nbsp;Create JP Bulletins</h3>
<p><br /></p>
@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
                <option class="dropdown-item" value="1">JobPosting</option>
                <option class="dropdown-item" value="2">Events</option>
            </select>
        <br/>
        <br/>
            <br />
            <br />
            @Html.ValidationMessageFor(model => model.BulletinCategoryEnum, "", new { @class = "text-danger" })
        </div>
            var textarea = document.getElementById("bulletinBody");
            if (openTag && styleTag) {
                let splitTag = openTag.split('>');
                splitTag[0] = splitTag[0] + ' style="' + styleTag+'"';
                splitTag[0] = splitTag[0] + ' style="' + styleTag + '"';
                openTag = splitTag.join(">");
            }
            if (textarea.selectionStart != undefined) {
        function handleFontSize(size) {
            const fontStyle = "font-size: " + size + "px";
            wrapText('<span>', '</span>', 'font-size: '+size+'px');
            wrapText('<span>', '</span>', 'font-size: ' + size + 'px');
        }
        // Close the dropdown menu if the user clicks outside of it

### Added Back to List Button
//////////////// Add Back to List button inline on JPChecklist Edit view ////////////////

<div class="row">
                    <div class="col-md-10 col-sm-3  col-xs-3 pt-20">
                        <input type="submit" value="Save" class="btn btn-default" />
                        <a href="../" class="btn btn-default inline">Back to List</a>
                    </div>
                </div>
            </div>

### Added Margins to View
////////////// Added Margins to JPChecklists Index view /////////////////////////

@model IEnumerable<JobPlacementDashboard.Models.JPChecklist>
<div class="container">
    <div class="col-sm-12">
        <table class="table" align="center">
            <tr>
                <th>
                <th></th>
            </tr>
@foreach (var item in Model) {
            @foreach (var item in Model)
            {
                <tr class="text-center">
                    <td>
                        @Html.DisplayFor(modelItem => item.ApplicationUser.FirstName)
                        @Html.DisplayFor(modelItem => item.JpRoundTables)
                    </td>
                    <td>
            <span class="font-20">@Html.ActionLink(HttpUtility.HtmlDecode("&#x270E;"), "Edit", new { id=item.JPChecklistid })</span> |         
            <span class="font-20">@Html.ActionLink(HttpUtility.HtmlDecode("&#x1F5D1;"), "Delete", new { id=item.JPChecklistid })</span>
                        <span class="font-20">@Html.ActionLink(HttpUtility.HtmlDecode("&#x270E;"), "Edit", new { id = item.JPChecklistid })</span> |
                        <span class="font-20">@Html.ActionLink(HttpUtility.HtmlDecode("&#x1F5D1;"), "Delete", new { id = item.JPChecklistid })</span>
                    </td>
                </tr>
            }
        </table>
    </div>
</div>

### Removed link from Index
//////////// Removed 'Create New' link from JPChecklists Index ///////////////////

@model IEnumerable<JobPlacementDashboard.Models.JPChecklist>
<p>
    @Html.ActionLink("Create New", "Create")
</p>
<table class="table" align="center">
    <tr>
        <th>

### Edit Student Drop Down
////// Edit Student Drop Down "Create an Account" to "Create a JP Account"  ////////////////////

                  <ul class="sub-menu">
                                <li class="job-placement-dropdown-item">Student Dashboard</li>
                                <li class="JPDropDown">@Html.ActionLink("Home", "Analytics", "Analytics")</li>
                                <li class="JPDropDown">@Html.ActionLink("Create an Account", "Create", "JPStudents")</li>
                                <li class="JPDropDown">@Html.ActionLink("Create a JP Account", "Create", "JPStudents")</li>
                                <li class="JPDropDown">@Html.ActionLink("Networking", "Networking", "Networking")</li>
                                <li class="JPDropDown">@Html.ActionLink("Bulletin", "Index", "JPBulletins")</li>
                                @if (User.Identity.IsAuthenticated)
                                {
                                <li class="JPDropDown">
                                    <a href="#">Log Data</a>
                                    <ul class="sub-menu">







    


