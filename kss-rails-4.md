## What is KSS?
The [KSS (Knyle Style Sheets)](http://warpspire.com/kss/) Documentation Methodology is a way of writing organized CSS for your web apps. With this methodology in place, you can build a "Living Styleguide" to help you and your team seeing the CSS you build. This methodology is used at [Github](https://github.com/styleguide/css) and is going to be used in my projects as well.

## What is a Living Styleguide?
A living styleguide is a live graphical representation, on one or more pages, of your web components and the changes you made to them inside your CSS source code. 

For example, you can see all the states of a button, at one place, with a living styleguide. Very helpful!

## The Setup  
What you need:

 * ruby 2.1.3  
 * rails 4.1.6  
 * kss 0.5.0  

## The Rails 4 Integration
In your `Gemfile`, add the following:  
    
    gem 'kss'
    
Now in your terminal, you can execute the following command to install KSS:
    
    bundle install
    
### The Controller
Next, we will need a controller for displaying the static pages. I use a controller named `pages` to do that. If you don't know what a Rails controller is all about, maybe you are not advanced enough to understand how to integrate this methodology into Rails. With all my respect, if this is the case for you, you should learn a little bit more about Rails before continue reading.

In your controller, add the following:
    
    def styleguide
      @styleguide ||= Kss::Parser.new(File.expand_path('app/assets/stylesheets', 
                                                       Rails.root))
    end
    helper_method :styleguide
    
This is going to tell KSS to scan your style sheets folder for KSS documentations.
### The Route
Now that we have a controller, we need a route. In your `config/routes.rb`, add the following:
   
    // Here, I use a controller named pages
    get 'styleguide' => 'pages#styleguide'
    
### The View
Next, we need to define a view. Create the following view inside `app/views/pages`:

`styleguide.html.erb`
    
    <%= styleguide_block '1.1' do -%>
      <button class="$modifier_class" type="button">Example Button</button>
    <% end -%>
    <%= javascript_include_tag 'kss' %>
    
This view is going to call a helper method called `styleguide_block` with the argument `1.1`. Once called, the helper method is going to render the HTML styleguide_block for the section 1.1 of your style sheets and use the button you provide as a model.

### The Javascript
KSS need a little bit of Javascript to operate. You can take a further look at the code to understand what it does but for now you need to download this file into your `vendor/assets/javascript/` folder:

  * [kss.js](https://raw.githubusercontent.com/kneath/kss/master/example/public/javascripts/kss.js)

We need to tell Rails about the new file.

Inside `config/initializers/assets.rb` add the following:
    
    Rails.application.config.assets.precompile += %w( kss.js )
    

### The Helper Method
Now it's time to create the helper method.

Inside `app/helpers/pages_helper.rb` add the following:
    
    def styleguide_block(section, &block)
      raise ArgumentError, "Missing block" unless block_given?

      @section = styleguide.section(section)

      if !@section.raw
        raise "KSS styleguide section is nil, is section '#{section}' defined in your css?"
      end

      content = capture(&block)
      render 'shared/styleguide_block', :section => @section, :example_html => content
    end
    
This helper method render a view named styleguide_block.
### The styleguide_block View
Every KSS Section in your style sheets are going to be render by this view. This is a "block". You are going to understand better what a block is after you see for the first time your living styleguide. But for now ...

Inside `app/views/shared` create the following view:

`_styleguide_block.html.erb`
    
    <div class="styleguide-example">
      <h3><%= section.section %> <em><%= section.filename %></em></h3>
      <div class="styleguide-description">
        <p><%= section.description %></p>
        <% if section.modifiers.any? %>
          <ul class="styleguide-modifier">
            <% section.modifiers.each do |modifier| %>
              <li><strong><%= modifier.name %></strong> - <%= modifier.description %></li>
            <% end %>
          </ul>
        <% end %>
      </div>
      <div class="styleguide-element">
        <%= example_html.html_safe %>
      </div>
      <pre><code data-language="html"><%= example_html.strip %></code></pre>
      <% section.modifiers.each do |modifier| %>
        <div class="styleguide-element styleguide-modifier">
          <span class="styleguide-modifier-name"><%= modifier.name %></span>
          <%= example_html.gsub('$modifier_class', modifier.class_name).html_safe %>
        </div>
        <pre><code data-language="html"><%= example_html.gsub('$modifier_class', modifier.class_name).strip %></code></pre>
      <% end %>
    </div>
    

### The CSS
Now that we have everything in place, we can add our CSS to start building our Living Styleguide. There is a lot of way on how to organize your style sheets. For me, I use this kind of structure:
    
    stylesheets
    |--components
    |   |--buttons.css.scss
    |   |--nav.css.scss
    |   |--...
    |--generic
    |   |--01-reset.css.scss
    |   |--02-typography.css.scss
    |   |--03-elements.css.scss
    |   |--...
    |--...
    
For this example, I create the `app/assets/stylesheets/components/buttons.css.scss` file with the following:
    
    /*
    Your standard form button.

    :hover    - Highlights when hovering.
    :disabled - Dims the button when disabled.

    Styleguide 1.1
    */
    button {
      padding: 5px 15px;
      line-height: normal;
      font-family: "Helvetica Neue", Helvetica;
      font-size: 12px;
      font-weight: bold;
      color: #666;
      text-shadow: 0 1px rgba(255, 255, 255, 0.9);
      border-radius: 3px;
      border: 1px solid #ddd;
      border-bottom-color: #bbb;
      background: #f5f5f5;
      filter: progid:DXImageTransform.Microsoft.gradient(GradientType=0, startColorstr='$start', endColorstr='$end');
      background: -webkit-gradient(linear, left top, left bottom, from(#f5f5f5), to(#e5e5e5));
      background: -moz-linear-gradient(top, #f5f5f5, #e5e5e5);
      box-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
      cursor: pointer;
    }
    button:disabled {
      opacity: 0.5;
    }
    
The CSS code above is a Section for the KSS Documentation Methodology.

For further reading about KSS, I suggest to you to read the [KSS Spec](https://github.com/kneath/kss/blob/master/SPEC.md).

### The Living Styleguide "LIVE"

Now you can start your Rails server, go to [http://localhost:3000/styleguide](http://localhost:3000/styleguide) and it should work. 

For a better presentation,  I suggest you use the following files to start:

  * [kss.css.scss](https://raw.githubusercontent.com/jimbeaudoin/kss-rails-example/master/app/assets/stylesheets/kss.css.scss)
  * [blackboard.css.scss](https://raw.githubusercontent.com/jimbeaudoin/kss-rails-example/master/app/assets/stylesheets/blackboard.css.scss)
  * [rainbow.js](https://raw.githubusercontent.com/jimbeaudoin/kss-rails-example/master/vendor/assets/javascripts/rainbow.js)

Put those files inside `app/assets/stylesheets` for SCSS and `vendor/assets/js` for Javascript:

Inside `config/initializers/assets.rb` add the following:
    
    Rails.application.config.assets.precompile += %w( rainbow.js )

Inside `app/views/pages/styleguide.html.erb` add the following at the bottom:
    
    <%= javascript_include_tag 'rainbow' %>
    
For the compilation of the assets to take place, you have to restart the server.

Once restarted, the Living Styleguide is going to be much more beautiful.

### The Source Code
You can find the source code of this basic integration at this Github Repository:

 * [jimbeaudoin/kss-rails-example](https://github.com/jimbeaudoin/kss-rails-example)

I hope this How-To is going to help you.

Thank You and Have a Nice Day!

[Jimmy](https://twitter.com/jim_beaudoin)

Thanks to the [kss-rails](https://github.com/dewski/kss-rails) project for code example!
