# TheatreCMS3
The C# and .NET Framework Live Project at the Tech Academy.

## Introduction
TheatreCMS3 is the interactive website for managing the content and productions for a theater/ acting company. The project is meant to be a content management service, for users who want to easily manage what displays on their website, without needing too much technical knowledge. For this project, my team worked on the blog portion of the website, and I was chosen to work on the comments section of the blog area. Throughout this process, I worked on my problem solving skills, breaking down large tasks into smaller tasks to be completed one at a time. I got experience incorporating ajax into a C# project. I 

### Stories
* [Create comment Model and Crud pages](#create-comment-model-and-crud-pages)
* [Styling Comments Page](#styling-comments-page)
* [Like and Dislike Button Implementation](#like-and-dislike-button-implementation)
* [Like Dislike Ratio Bar](#like-dislike-ratio-bar)
* [Delete Button](#delete-button)

### Create Comment Model and Crud Pages
The first task was creating the model for the comment itself, and then use Visual Studio to scaffold the Crud pages. As I went through the project, the comment model was updated to this iteration. The constructor is set up to set the comment date to the current time, representing the time it was created. One roadblock I had was getting the author property to be saved as the current application user. I attempted to do this in the constructor at first, but ended up solving the problem by adding some code into the create method in the controller. 

``` 
    public Comment()
    {
        CommentDate = DateTime.Now;
    }

    [Key]
    public int CommentId { get; set; }
    public virtual ApplicationUser Author { get; set; }
    public string Message { get; set; }
    public DateTime CommentDate { get; set; }
    public int Likes { get; set; }
    public int Dislikes { get; set; }

    public double LikeRatio()
    {
        // if you don't explicitly convert to double, it will return zero.
        double likeRatio = Math.Round((double)Likes / ((double)Likes + (double)Dislikes) * 100.00);
        return likeRatio;
    }
```
```
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "Author,CommentId,Message,CommentDate,Likes,Dislikes")] Comment comment)
        {
            if (ModelState.IsValid)
            {
                string userId = HttpContext.User.Identity.GetUserId();
                comment.Author = db.Users.FirstOrDefault(x => x.Id == userId);
                db.Comment.Add(comment);
                db.SaveChanges();
                return RedirectToAction("Index");
            }

            return View(comment);
        }
```

    
### Styling Comments Page
After creating the model for comments, and scaffolding some crud pages, I was tasked with creating a partial view for comments that could be rendered on any page. Using the provided color scheme, here is the basic comment page I came up with to display data from all the comments in the database, starting with the most recent.   

```
    @foreach (var item in Model.OrderByDescending(i => i.CommentDate))
    {
        <div class="col-lg-12 p-2 cms-bg-main-light border border-dark">
          <div class="row">
            <a class="col-2" href="">@Html.DisplayFor(modelItem => item.Author.UserName)</a>
            <p class="col-8 border border-light cms-text-dark">@Html.DisplayFor(modelItem => item.Message)</p>
            <button class="col-1 p-1 cms-text-dark btn">
              <i class="fa fa-trash" aria-hidden="true"></i>
            </button>
          </div>
          <div class="row">
            <p class="col-2">@item.TimeSincePost()</p>

            <button class="btn btn-outline-light cms-bg-main">Like</button>
            <p class="col-1">@Html.DisplayFor(modelItem => item.Likes)</p>
            <button class="btn btn-outline-light cms-bg-main">Dislike</button>
            <p class="col-1">@Html.DisplayFor(modelItem => item.Dislikes)</p>
            <button class="btn btn-outline-light cms-bg-main">Reply</button>

          </div>
        </div>
```
![image](https://user-images.githubusercontent.com/80490144/126914973-43e51ebf-c759-47f5-8393-fa72ab9fde09.png)

### Like and Dislike Button Implementation
After styling the comments page, I was tasked with adding asynchronous functionality to the like and dislike buttons. There is one cool way to do this, in C# without any javascript, but it would have required a package outside the scope of this project, so I did it with corresponding like and dislike functions, the like functions are shown here. At first because of the foreach loop and multiple comment boxes being generated, I was targeting the id of all the comment boxes that were being created. To target only the comment clicked, I generated unique html ids to target each separate comment element. I did this by generating a comment id in C# that would be unique to each comment,
```
string likesId = item.CommentId + "likes";
<p id="@likesId" class="col">@Html.DisplayFor(modelItem => item.Likes)</p>
```
and then recreating it in the javasript as the same name. 
```
function addLike(commentId) {
    {
        try {
            var likesId = commentId + "likes";
            var progressBarId = commentId + "bar";
            var ratioId = "#" + commentId + "ratio";
            $.ajax({
                url: onLikeClickUrl,
                type: "POST",
                data: { id: commentId },
                success: function (result) {
                    document.getElementById(likesId).innerHTML = result.Data["likes"];
                    $(ratioId).load(location.href + " " + ratioId);
                    document.getElementById(progressBarId).style.width = result.Data["likeRatio"];
                },
                error: function (error) {
                    alert(error);
                }
            });
        }
        catch (e) {
            alert(e.message);
        }
    }
}
```
```
      [HttpPost]
      public JsonResult OnLikeClick(int Id)
      {
          Comment comment = db.Comment.Find(Id);
          var result = new JsonResult();
          comment.Likes++;
          db.Entry(comment).State = EntityState.Modified;
          db.SaveChanges();
          result.Data = comment.Likes;
          return Json(result, JsonRequestBehavior.AllowGet);
      }
```
 
### Like Dislike Ratio Bar
This was a challenging one. I was tasked with adding a like dislike ratio bar that updates asynchronously when like or dislike is clicked.
![image](https://user-images.githubusercontent.com/80490144/126916063-51bae6b5-1d13-49f9-a892-9d9c886758a7.png)
For the progress bar, I had to generate a unique Id as well and recreate it in the javascript to target the element. To update the element displaying the percentages,
```
<p id="@ratioId" class="col p-0 m-0 cms-bg-dark">@item.LikeRatio()% Likes | @dislikeRatio% Dislikes</p>
```
I added this line of code to my like and dislike ajax, respectively.
```
$(ratioId).load(location.href + " " + ratioId);
```
However, when trying .load on this element, (the progress bar itself)
```
<div id="@progressBarId" class="progress-bar bg-success" role="progressbar" aria-valuenow="@item.LikeRatio()" aria-valuemin="0" aria-valuemax="100" style="width: @item.LikeRatio()%; "></div>
```
The style element was not updating. To fix this problem, I updated my dislike controller method to send the updated likeRatio, with a percent symbol concatenated on to it. 
```
        db.SaveChanges();
        result.Data = new { likes = comment.Likes, likeRatio = (Int32)comment.LikeRatio() + "%" };
        return Json(result, JsonRequestBehavior.AllowGet);
```
And then used it in my ajax method, to style the width directly. 
```
        document.getElementById(progressBarId).style.width = result.Data["likeRatio"];
```

### Delete Button
For my final story, I was tasked with updating the delete functionality. I created a modal that pops up when delete is clicked, and asks the user to confirm deletion of the comment, Upon confirm, the modal disappears, the comment is deleted and a confirmation message displays for three seconds, and then fades away. 
Here is the JS:
```
$((function() {
    var url;
    var redirectUrl;
    var target;
    var commentContainerId;
    $('#blog-comments-container').prepend(`
        <div class="blog-comments-deleteSuccessBarContainer w-50 cmg-bg-dark">
            <div class="blog-comments-deleteSuccessBar progress ">
              <div class="progress-bar bg-success" role="progressbar" style="width: 100%" aria-valuenow="100" aria-valuemin="0" aria-valuemax="100">
                    <p class="my-auto">The comment was deleted successfully<i class="fa fa-check" aria-hidden="true"></i></p></div>
            </div>
        </div>
            `);
    $('.blog-comments-deleteSuccessBarContainer').hide();
    $('body').append(`
        <div id="deleteModal" class="modal" tabindex="-1" role="dialog">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title cms-text-main">Delete Comment?</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <p class="delete-modal-body cms-text-secondary-dark">Modal body text goes here.</p>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" id="confirm-delete">Confirm Delete</button>
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    </div>
                </div>
            </div>
        </div>
`);

    $(".delete-comment").on('click', (e) => {
            e.preventDefault();

            target = e.target;
            
            var Id = $(target).data('id');
            var controller = $(target).data('controller');
            var action = $(target).data('action');
            var bodyMessage = $(target).data('body-message');
            commentContainerId = "#" + Id + "comment";
            redirectUrl = $(target).data('redirect-url');

            url = "/Blog/" + controller + "/" + action + "/" + Id;
            $(".delete-modal-body").text(bodyMessage);
            $("#deleteModal").modal("show");
        });

    $("#confirm-delete").on('click', () => {
        $("#deleteModal").modal("hide");
        try {  
            $.ajax({
                url: url,
                type: "POST",
                success: function (result) {
                    $(".blog-comments-deleteSuccessBarContainer").show().delay(3000).fadeOut(1500);
                    $(commentContainerId).hide();      
                },
                error: function (error) {
                    alert(error);
                }
            });
        }
        catch (e) {
            alert(e.message);
        }

        });
    }()));
```    
 
