<h1>Custom Events in Laravel</h1>
<p>Assume that you want to send the email notification, as a security measure, when someone logs into the application. How can you send this email?</p>
<p>Recall, any service bind with service container can be available to use via a facade. These two communicate each other through the service container with a binding key.</p>
<p>But in Event listener phenomena, Event class and Listener class are bonded directly with each other in EventServiceProvider.php by $listen array. Here, the key is the event and value is the listener.</p>
<h2>Event Listener creating steps:</h2>
<p>1. Open <code>app/Providers/EventServiceProvider.php</code> file and register our custom event and listener mappings.</p>
<p>2. Run the command: <code>php artisan event:generate</code></p>
<p>3. Update the <code>app/Events/ClearCache.php</code> class to give any property or methods (ex. <code>public $cache_keys = [];</code>)</p>
<p>4. Update the handle method of your custom listener to work with the property or method created in step 2.</p>
<p>5. Create a controller which will invoke <code>event</code> helper with an insance of your custom event</p>
<h1>Understanding of Event Listener</h1>
<p>Here is the <code>app/Providers/EventServiceProvider.php</code><p>
<blockquote>
<?php<br/>
namespace App\Providers;<br/>
use Illuminate\Support\Facades\Event;<br/>
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;<br/>
class EventServiceProvider extends ServiceProvider<br/>
{<br/>
    /**<br/>
     * The event listener mappings for the application.<br/>
     *<br/>
     * @var array<br/>
     */<br/>
    protected $listen = [<br/>
        'App\Events\Event' => [<br/>
        'App\Listeners\EventListener',<br/>
        ],<br/>
    ];<br/>
    /**<br/>
     * Register any events for your application.<br/>
     *<br/>
     * @return void<br/>
     */<br/>
    public function boot()<br/>
    {<br/>
        parent::boot();<br/>
        //<br/>
    }<br/>
}
</blockquote>
<p>Now, <code>Illuminate\Auth\Events\Login</code> is an event that'll be raised when someone logs into an application. We'll bound that event to the <code>App\Listeners\SendEmailNotification</code> listener, so it'll be triggered on the login event.</p>
<blockquote>
<?php<br/>
namespace App\Providers;<br/>
use Illuminate\Support\Facades\Event;<br/>
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;<br/>
class EventServiceProvider extends ServiceProvider<br/>
{<br/>
    /**<br/>
     * The event listener mappings for the application.<br/>
     *<br/>
     * @var array<br/>
     */<br/>
    protected $listen = [<br/>
        'Illuminate\Auth\Events\Login' => [<br/>
        'App\Listeners\SendEmailNotification',<br/>
        ],<br/>
    ];<br/>
    /**<br/>
     * Register any events for your application.<br/>
     *<br/>
     * @return void<br/>
     */<br/>
    public function boot()<br/>
    {<br/>
        parent::boot();<br/>
        //<br/>
    }<br/>
}
</blockquote>
<p>After populating $listen array with Event Listener, we will run the following command:</p>
<blockquote>
php artisan event:generate
</blockquote>
<p>This command generates event and listener classes listed under the $listen property if they do not exist.</p>
<p>In our case, the <code>Illuminate\Auth\Events\Login</code> event already exists, so it only creates the <code>App\Listeners\SendEmailNotification</code> listener class. In fact, it would have created the <code>Illuminate\Auth\Events\Login</code> event class too if it didn't exist.</p>
<p>Let's have a look at the listener class created at <code>app/Listeners/SendEmailNotification.php</code>.</p>
<blockquote>
<?php<br/>
namespace App\Listeners;<br/>
use Illuminate\Auth\Events\Login;<br/>
use Illuminate\Queue\InteractsWithQueue;<br/>
use Illuminate\Contracts\Queue\ShouldQueue;<br/>
class SendEmailNotification<br/>
{<br/>
    /**<br/>
     * Create the event listener.<br/>
     *<br/>
     * @return void<br/>
     */<br/>
    public function __construct()<br/>
    {<br/>
        //<br/>
    }<br/>
    /**<br/>
     * Handle the event.<br/>
     *<br/>
     * @param  Login  $event<br/>
     * @return void<br/>
     */<br/>
    public function handle(Login $event)<br/>
    {<br/>
         <br/>
    }<br/>
}
</blockquote>
<p><code>handle</code> method that will be invoked with appropriate dependencies whenever the listener is triggered.</p>
<h1>Create a Custom Event</h1>
<p>Consider, our application needs to clear caches in our system at certain points. We'll raise the <code>CacheClear</code> event.</p>
<p>Open <code>app/Providers/EventServiceProvider.php</code> file and register our custom event and listener mappings.</p>
<blockquote><?php<br/>
namespace App\Providers;<br/>
use Illuminate\Support\Facades\Event;<br/>
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;<br/>
class EventServiceProvider extends ServiceProvider<br/>
{<br/>
    /**<br/>
     * The event listener mappings for the application.<br/>
     *<br/>
     * @var array<br/>
     */<br/>
    protected $listen = [<br/>
        'App\Events\ClearCache' => [<br/>
        'App\Listeners\WarmUpCache',<br/>
        ],<br/>
    ];<br/>
    /**<br/>
     * Register any events for your application.<br/>
     *<br/>
     * @return void<br/>
     */<br/>
    public function boot()<br/>
    {<br/>
        parent::boot();<br/>
        //<br/>
    }<br/>
}
</blockquote>
<p>Run the command:</p>
<blockquote>php artisan event:generate</blockquote>
<p>With a few changes, the app/Events/ClearCache.php class should look like this:</p>
<blockquote>
<?php<br/>
namespace App\Events;<br/>
use Illuminate\Broadcasting\Channel;<br/>
use Illuminate\Queue\SerializesModels;<br/>
use Illuminate\Broadcasting\PrivateChannel;<br/>
use Illuminate\Broadcasting\PresenceChannel;<br/>
use Illuminate\Foundation\Events\Dispatchable;<br/>
use Illuminate\Broadcasting\InteractsWithSockets;<br/>
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;<br/>
class ClearCache<br/>
{<br/>
    use Dispatchable, InteractsWithSockets, SerializesModels;<br/>
     <br/>
    public $cache_keys = [];<br/>
    /**<br/>
     * Create a new event instance.<br/>
     *<br/>
     * @return void<br/>
     */<br/>
    public function __construct(Array $cache_keys)<br/>
    {<br/>
        $this->cache_keys = $cache_keys;<br/>
    }<br/>
    /**<br/>
     * Get the channels the event should broadcast on.<br/>
     *<br/>
     * @return Channel|array<br/>
     */<br/>
    public function broadcastOn()<br/>
    {<br/>
        return new PrivateChannel('channel-name');<br/>
    }<br/>
}
</blockquote>
<p>Next, let's have a look at the listener class with an updated handle method at <code>app/Listeners/WarmUpCache.php</code>.</p>
<blockquote>
<?php<br/>
namespace App\Listeners;<br/>
use App\Events\ClearCache;<br/>
use Illuminate\Queue\InteractsWithQueue;<br/>
use Illuminate\Contracts\Queue\ShouldQueue;<br/>
class WarmUpCache<br/>
{<br/>
    /**<br/>
     * Create the event listener.<br/>
     *<br/>
     * @return void<br/>
     */<br/>
    public function __construct()<br/>
    {<br/>
        //<br/>
    }<br/>
    /**<br/>
     * Handle the event.<br/>
     *<br/>
     * @param  ClearCache  $event<br/>
     * @return void<br/>
     */<br/>
    public function handle(ClearCache $event)<br/>
    {<br/>
        if (isset($event->cache_keys) && count($event->cache_keys)) {<br/>
            foreach ($event->cache_keys as $cache_key) {<br/>
                // generate cache for this key<br/>
                // warm_up_cache($cache_key)<br/>
            }<br/>
        }<br/>
    }<br/>
}
</blockquote>
<p>Now, create a controller file at <code>app/Http/Controllers/EventController.php</code> to demonstrate how you could raise an event.</p>
<blockquote>
<?php<br/>
namespace App\Http\Controllers;<br/>
use App\Http\Controllers\Controller;<br/>
use App\Library\Services\Contracts\CustomServiceInterface;<br/>
use App\Post;<br/>
use Illuminate\Support\Facades\Gate;<br/>
use App\Events\ClearCache;<br/>
class EventController extends Controller<br/>
{<br/>
    public function index()<br/>
    {<br/>
        // ...<br/>
         <br/>
        // you clear specific caches at this stage<br/>
        $arr_caches = ['categories', 'products'];<br/>
         <br/>
        // want to raise ClearCache event<br/>
        event(new ClearCache($arr_caches));<br/>
         <br/>
        // ...<br/>
    }<br/>
}
</blockquote>
<p>The <code>event</code> helper function is used to raise an event from anywhere within an application.</p>
<h1>What Is an Event Subscriber?</h1>
<p>Whenever you want to logically group event listeners, you have to choose Event Subscribers</p>
<blockquote>
<?php<br/>
// app/Listeners/ExampleEventSubscriber.php<br/>
namespace App\Listeners;<br/>
class ExampleEventSubscriber<br/>
{<br/>
    /**<br/>
     * Handle user login events.<br/>
     */<br/>
    public function sendEmailNotification($event) {<br/>
        // get logged in username<br/>
        $email = $event->user->email;<br/>
        $username = $event->user->name;<br/>
         <br/>
        // send email notification about login...<br/>
    }<br/>
    /**<br/>
     * Handle user logout events.<br/>
     */<br/>
    public function warmUpCache($event) {<br/>
        if (isset($event->cache_keys) && count($event->cache_keys)) {<br/>
            foreach ($event->cache_keys as $cache_key) {<br/>
                // generate cache for this key<br/>
                // warm_up_cache($cache_key)<br/>
            }<br/>
        }<br/>
    }<br/>
    /**<br/>
     * Register the listeners for the subscriber.<br/>
     *<br/>
     * @param  Illuminate\Events\Dispatcher  $events<br/>
     */<br/>
    public function subscribe($events)<br/>
    {<br/>
        $events->listen(<br/>
            'Illuminate\Auth\Events\Login',<br/>
            'App\Listeners\ExampleEventSubscriber@sendEmailNotification'<br/>
        );<br/>
         <br/>
        $events->listen(<br/>
            'App\Events\ClearCache',<br/>
            'App\Listeners\ExampleEventSubscriber@warmUpCache'<br/>
        );<br/>
    }<br/>
}
</blockquote>
<p>It's the <code>subscribe</code> method that is responsible for registering listeners. The first argument of the subscribe method is the instance of the <code>Illuminate\Events\Dispatcher</code> class that you could use to bind events with listeners using the listen method.</p>
<blockquote>
<?php<br/>
namespace App\Providers;<br/>
use Illuminate\Support\Facades\Event;<br/>
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;<br/>
class EventServiceProvider extends ServiceProvider<br/>
{<br/>
    /**<br/>
     * The subscriber classes to register.<br/>
     *<br/>
     * @var array<br/>
     */<br/>
    protected $subscribe = [<br/>
        'App\Listeners\ExampleEventSubscriber',<br/>
    ];<br/>
    /**<br/>
     * Register any events for your application.<br/>
     *<br/>
     * @return void<br/>
     */<br/>
    public function boot()<br/>
    {<br/>
        parent::boot();<br/>
        //<br/>
    }<br/>
}
</blockquote>
