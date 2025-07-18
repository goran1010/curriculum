### Introduction

These projects will give you a chance to actually build some forms, both using nearly-pure HTML and then graduating to using the helper methods that Rails provides.

### Project: Bare metal forms and helpers

In this project, you'll build a form the old fashioned way and then the Rails way.

### Assignment

<div class="lesson-content__panel" markdown="1">

#### Set up the Back end

You'll get good at setting up apps quickly in the coming lessons by using more or less this same series of steps (though we'll help you less and less each time):

1. Build a new rails app (called "re-former").
1. Create a new Github repo and connect the remote to your local git repo. Check in and commit the initial stuff.
1. Modify your README file to say something you'll remember later, like "This is part of the Forms Project in The Odin Project's Ruby on Rails Curriculum. Find it at [https://www.theodinproject.com](https://www.theodinproject.com)"
1. Create and migrate a User model with `:username`, `:email` and `:password`.
1. Add validations for presence to each field in the model.
1. Create the `:users` resource in your routes file so requests actually have somewhere to go.  Use the `only:` option to specify just the `:new` and `:create` actions.
1. Build a new UsersController (either manually or via the `$ rails generate controller Users` generator).
1. Write empty methods for `#new` and `#create` in your UsersController.
1. Create your `#new` view in `app/views/users/new.html.erb`.
1. Fire up a rails server in another tab.
1. Make sure everything works by visiting `http://localhost:3000/users/new` in the browser.

#### HTML form

The first form you build will be mostly HTML (remember that stuff at all?).  Build it in your New view at `app/views/users/new.html.erb`.  The goal is to build a form that is almost identical to what you'd get by using a Rails helper so you can see how it's done behind the scenes.

1. Build a form for creating a new user. See the [W3Schools page for forms](https://www.w3schools.com/tags/tag_form.asp) if you’ve totally forgotten how they work. Specify the `method` and the `action` attributes in your `<form>` tag (use `$ rails routes` to see which HTTP method and path are being expected based on the resource you created).  Include the attribute `accept-charset="UTF-8"` as well, which Rails naturally adds to its forms to specify Unicode character encoding.

1. Create the proper input tags for your user's fields (email, username and password).  Use the proper password input for "password".  Be sure to specify the `name` attribute for these inputs.  Make label tags which correspond to each field.

1. For CSRF safety with Rails 7, Turbo is enabled by default in new apps. Turbo intercepts form submission and makes a partial XHR request instead of a standard HTTP request with full page reload. To get a better grasp of Rails protection against [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery), let's take a short detour and disable Turbo for this form by setting the data attribute `data-turbo=false`.
In the dev tools network tab, compare the request type with and without the `data-turbo=false` attribute to confirm it works as expected.

1. Submit your form and view the server output. The request should be intercepted before reaching your controller and the server will throw a CSRF error `ActionController::InvalidAuthenticityToken  (Can't verify CSRF token authenticity.)`.

    That's because Rails by default automatically protects you from [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) and it requires you to verify that the form was actually submitted from a page you generated. In order to do so, it generates an ["authenticity token"](http://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf) which looks like gibberish but helps Rails match the form with your session and the application.

   So, if you want to create your own form that gets handled by Rails, you need to provide the token somehow as well. Luckily, Rails gives you a method called `form_authenticity_token` to do so

   ```erb
   <input
   type="hidden"
   name="authenticity_token"
   value="<%= form_authenticity_token %>"
   >
   ```

   You'll now notice the token in the server output:

   ```bash
   ...
   Parameters: {"utf8"=>"✓", "authenticity_token"=>"jJa87aK1OpXfjojryBk2Db6thv0K3bSZeYTuW8hF4Ns=", "email"=>"foo@bar.com", "commit"=>"Submit Form"}
   ```

1. However, if you look at the server output, you will see nothing much happening after the log of parameters received. The log should indicate a completed response with status 204 (no content). And indeed, if you look at the network tab in your inspector, you can see that a request was issued, but a response of `204 No Content` is returned.
1. That's A-OK because it means that we've successfully gotten through our blank `#create` action in the controller (and didn't specify what should happen next).  Look at the server output.  It should include the parameters that were submitted, looking something like:

   ```bash
   Started POST "/users" for 127.0.0.1 at 2013-12-12 13:04:19 -0800
   Processing by UsersController#create as TURBO_STREAM
   Parameters: {"authenticity_token"=>"WUaJBOpLhFo3Mt2vlEmPQ93zMv53sDk6WFzZ2YJJQ0M=", "username"=>"foobar", "email"=>"foo@bar.com", "password"=>"[FILTERED]"}
   ```

That looks a whole lot like what you normally see when Rails does it, right?

#### Controller setup

1. Go into your UsersController and build out the `#create` action to take those parameters and create a new User from them.  If you successfully save the user, you should redirect back to the New User form (which will be blank) and if you don't, it should render the `:new` form again (but it will still have the existing information entered in it).  You should be able to use something like:

   ```ruby
   # app/controllers/users_controller.rb
   def create
     @user = User.new(username: params[:username], email: params[:email], password: params[:password])

     if @user.save
       redirect_to new_user_path
     else
       render :new, status: :unprocessable_entity
     end
   end
   ```

1. Test this out -- can you now create users with your form? If so, you should see an INSERT SQL command in the server log.
1. We're not done just yet... that looks too long and difficult to build a user with all those `params` calls. It'd be a whole lot easier if we could just use a hash of the user's attributes so we could just say something like `User.new(user_params)`. Let's build it... we need our form to submit a hash of attributes that will be used to create a user, just like we would with Rails' `form_with` method. Remember, that method submits a top level `user` field which actually points to a hash of values. This is easy to achieve, though -- just change the `name` attribute slightly. Nest your three User fields inside the variable attribute using brackets in their names, e.g. `name="user[email]"`.
1. Resubmit. Now your user parameters should be nested under the `"user"` key like:

   ```bash
   Parameters: {"authenticity_token" => "WUaJBOpLhFo3Mt2vlEmPQ93zMv53sDk6WFzZ2YJJQ0M=", "user" =>{ "username" => "foobar", "email" => "foo@bar.com", "password" => "[FILTERED]" } }
   ```

1. You'll get some errors because now your controller will need to change.  But recall that we're no longer allowed to just directly call `params[:user]` because that would return a hash and Rails' security features prevent us from doing that without first validating it.
1. Go into your controller and comment out the line in your `#create` action where you instantiated a `::new` User (we'll use it later).
1. Implement a private method at the bottom called `user_params` which will `expect` the proper fields (see the [Controllers Lesson](/lessons/ruby-on-rails-controllers) for a refresher).
1. Add a new `::new` User line which makes use of that new allow params method.
1. Submit your form now.  It should work marvelously (once you debug your typos)!

#### Railsy forms with #form_tag

Now we'll start morphing our form into a full Rails form using the `#form_tag` and `#*_tag` helpers.  There's actually very little additional help that's going on and you'll find that you're mostly just renaming HTML tags into Rails tags.

1. Comment out your entire HTML form.  It may be helpful to save it for later on if you get stuck.
1. Convert your `<form>` tag to use a `#form_tag` helper and all of your inputs into the proper helper tags via `#*_tag` methods.  The good thing is that you no longer need the authentication token because Rails will insert that for you automatically. `#form_tag` is soft-deprecated as stated in the current Rails Guide. Have a look at the older documentation for [Action View Form Helpers](https://guides.rubyonrails.org/v5.2/form_helpers.html).
1. See the [Form Tag API Documentation](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-form_tag) for a list and usage of all the input methods you can use with `#form_tag`.
1. Test out your form.  You'll need to change your `#create` method in the controller to once again accept normal top level User attributes, so uncomment the old `User.new` line and comment out the newer one.
1. You've just finished the first step.

#### Turn Turbo back ON

Above, we asked to disable Turbo for the sake of the exercise.

1. Re-enable form submission with Turbo by removing the `data-turbo=false` attribute on the form tag, then also remove the hidden input with CSRF token tag and submit.

   No more CSRF error?!

1. The form is now submitted with Turbo, yet Rails still protects you by verifying a CSRF token. Where does this token comes from? Check your inspector and your `application.html.erb` template. Can you find a CSRF token that is always available? Remove this one too from `application.html.erb`, and verify that the server hits back with a CSRF error.

1. Reinstate the CSRF token tag in both places and carry on.

#### Railsy-er forms with #form_with

`#form_tag` probably didn't feel that useful -- it's about the same amount of work as using `<form>`, though it does take care of the authenticity token stuff for you.  Now we'll convert that into `#form_with`, which will make use of our model objects to build the form.

1. Modify your `#new` action in the controller to instantiate a blank User object and store it in an instance variable called `@user`.
1. Comment out your `#form_tag` form in the `app/views/users/new.html.erb` view (so now you should have TWO commented out form examples).
1. Rebuild the form using `#form_with` and the `@user` from your controller.  You'll need to switch your controller's `#create` method again to accept the nested `:user` hash from `params`.
1. Play with the `#input` method options -- add a default placeholder (like "<example@example.com>" for the email field), make it generate a different label than the default one (like "Your user name here"), and try starting with a value already populated.  Some of these things you may need to Google for, but check out the [`#form_with` Rails API docs](https://api.rubyonrails.org/v6.1.1/classes/ActionView/Helpers/FormHelper.html#method-i-form_with)
1. Test it out.

#### Editing

1. Update your routes and controller to handle editing an existing user.  You'll need your controller to find a user based on the submitted `params` ID.
1. Create the Edit view at `app/views/users/edit.html.erb` and copy/paste your form from the New view.  Your HTML and `#form_tag` forms (which should still be commented out) will not work -- they will submit the form as a POST request when you need it to be a PATCH (PUT) request (remember your `$ rails routes`?).  It's an easy fix, which you should be able to see if you attempt to edit a user with the `#form_with` form (which is smart enough to know if you're trying to edit a user or creating a new one).
1. Do a "view source" on the form generated by `#form_with` in your Edit view, paying particular attention to the hidden fields at the top nested inside the `<form>`.  See it?
1. Modify the top of your form view to display a list of the error messages that are attached to the failed model object when it fails validations. Recall the `#errors` and `#full_messages` methods.
1. Save this project to Git and upload to GitHub.

</div>
