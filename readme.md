## Installation

1. Clone this repository
2. `composer install`
3. `npm install` or `yarn install`
4. make a copy of `.env.example` and rename to `.env`
5. Create and setup pusher account
   ````
   login to https://pusher.com/ and create new app
   put pusher settings to .env file
6. in `.env` file
      ````   
        BROADCAST_DRIVER=pusher
        PUSHER_APP_ID=your_ID
        PUSHER_APP_KEY=your_key
        PUSHER_APP_SECRET=your_secret
        PUSHER_APP_CLUSTER=your_cluster
5. database name `laravel-chat` in `.env` file
6. `php artisan migrate`
7. `php artisan key:generate`
8. `php artisan serve`
9. `php artisan websockets:serve`

Now test it in your browser.


