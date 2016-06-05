**jrean/laravel-user-verification** is a PHP package built for Laravel 5.* to
easily handle a user verification and validate the e-mail.

[![Latest Stable Version](https://poser.pugx.org/jrean/laravel-user-verification/v/stable)](https://packagist.org/packages/jrean/laravel-user-verification) [![Total Downloads](https://poser.pugx.org/jrean/laravel-user-verification/downloads)](https://packagist.org/packages/jrean/laravel-user-verification) [![License](https://poser.pugx.org/jrean/laravel-user-verification/license)](https://packagist.org/packages/jrean/laravel-user-verification)

## About

- [x] Generate and store a verification token for a registered user
- [x] Send an e-mail with the verification token link
- [x] Handle the verification of the token
- [x] Set the user as verified

## Installation

This project can be installed via [Composer](http://getcomposer.org).
To get the latest version of Laravel User Verification, simply add the following line to
the require block of your composer.json file:

    {
        "require": {
                "jrean/laravel-user-verification": "^2.0"
        }

    }

You'll then need to run `composer install` or `composer update` to download the
package and have the autoloader updated.

Or run the following command:

    "composer require jrean/laravel-user-verification"


### Add the Service Provider

Once Larvel User Verification is installed, you need to register the service provider.

Open up `config/app.php` and add the following to the `providers` key:

* `Jrean\UserVerification\UserVerificationServiceProvider::class`

### Add the Facade/Alias

Open up `config/app.php` and add the following to the `aliases` key:

* `'UserVerification' => Jrean\UserVerification\Facades\UserVerification::class`

## Configuration

Prior to use this package the table representing the `User` must be updated with
two new columns, `verified` and `verification_token`.

**It is mandatory to add the two columns on the same table and where the user's
e-mail is stored.**

The model representing the `User` must implement the authenticatable
interface `Illuminate\Contracts\Auth\Authenticatable` which is the default with
the Eloquent `User` model.

### Migration

Generate the migration file with the following artisan command:

```
php artisan make:migration add_verification_to_:table_table --table=":table"
```

Where `:table` is replaced by the table name of your choice.

For instance if you want to keep the default Eloquent User table:

```
php artisan make:migration add_verification_to_users_table --table="users"
```

Once the migration is generated, edit the generated migration file in `database/migration` with the following:

```
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table(':table', function (Blueprint $table) {
            $table->boolean('verified')->default(false);
            $table->string('verification_token')->nullable();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table(':table', function (Blueprint $table) {
            $table->dropColumn('verified');
            $table->dropColumn('verification_token');
        });
    }

```

Where `:table` is replaced by the table name of your choice.

Migrate the migration with the following command:

```
php artisan migrate
```

### Exception

If the table representing the user is not updated and a user instance is given
to the package without implement the authenticatable interface
`Illuminate\Contracts\Auth\Authenticatable` a `ModelNotCompliantException` will
be thrown.


## E-mail

This package provides a method to send an e-mail with a link containing the verification token.

- `send(AuthenticatableContract $user, $subject = null, $from = null, $name =
    null)`

By default the package will use the `from` and `name` values defined into the
`config/mail.php` file:

    'from' => ['address' => '', 'name' => ''],

If you want to override the values, simply set the `$from` and (optional)
`$name` parameters.

Please refer to the Laravel [documentation](https://laravel.com/docs/) for the proper e-mail component configuration.

### E-mail View

The user will receive an e-mail with a link leading to the `getVerification()`
method (endpoint). Create a view for this e-mail at
`resources/views/emails/user-verification.blade.php`. The view will receive the
`$user` variable which contains the user details such as the verification
token. Here is a sample e-mail view content to get you started with:

```
Click here to verify your account: <a href="{{ $link = url('verification', $user->verification_token) . '?email=' . urlencode($user->email) }}"> {{ $link }}</a>
```

## Errors

This package throws several exceptions. You are free to use `try/catch`
statements or to rely on Laravel built-in exceptions handling.

* `ModelNotCompliantException`

The model instance provided is not compliant with this package. It must
implement the authenticatable interface
`Illuminate\Contracts\Auth\Authenticatable`

* `TokenMismatchException`

Wrong verification token.

* `UserIsVerifiedException`

The given user is already verified.

* `UserNotFoundException`

No user found for the given e-mail adresse.

### Error View

Create a view for the default verification error route `/verification/error` at
`resources/views/errors/user-verification.blade.php`. If an error occurs, the
user will be redirected to this route and this view will be rendered. Customize
this view to your needs.

## Usage

### API

The package public API offers three (3) methods.

* `generate(AuthenticatableContract $user)`

Generate and save a verification token for the given user.

* `send(AuthenticatableContract $user, $subject = null, $from = null, $name =
* null)`

Send by e-mail a link containing the verification token.

* `process($email, $token, $userTable)`

Process the token verification for the given e-mail and token.

### Facade

The package offers a facade callable with `UserVerification::`.

You can use it over the three (3) previous listed public methods.

### Trait

The package also offers two (2) traits for a quick implementation.

`Jrean\UserVerification\Traits\VerifiesUsers`

which includes:

`Jrean\UserVerification\Traits\RedirectsUsers`

#### Endpoints

The two following methods are endpoints you can join by defining the proper
route(s) of your choice.

* `getVerification(Request $request, $token)`

Handle the user verification. It requires a string parameter representing the
verification token to verify.

* `getVerificationError()`

Do something if the verification fails.

#### Custom attributes/properties

To customize the package behaviour and the redirects you can implement and
customize six (6) attributes/properties:

* `$redirectIfVerified = '/';`

Where to reditect if the authenticated user is already verified.

* `$redirectAfterTokenGeneration = '/';`

Where to redirect after a successful verification token generation.

* `$redirectAfterVerification = '/';`

Where to redirect after a successful verification token verification.

* `$redirectIfVerificationFails = '/verification/error';`

Where to redirect after a failling token verification.

* `$verificationErrorView = 'errors.user-verification';`

Name of the view returned by the getVerificationError method.

* `$userTable = 'users';`

Name of the default table used for managing users.

#### Custom methods

You can easily customize the package behaviour by overriding/overwriting the
public methods. Dig into the source.

## Guidelines

This package whishes to let you be creative while offering you a predefined
path. The following guidelines assume you have configured Laravel for the
package as well as created and migrated the migration according to this
documentation.

This package doesn't require the user to be authenticated to perform the
verification. You are free to implement any flow you may want to achieve.
Note that by default the behaviour of Laravel is to return an authenticated
user straight after the registration step.

### Example

The following code sample aims to showcase a quick and basic implementation.

Edit the `app\Http\Controller\Auth\AuthController.php` file. It is highly
recommended to read the content of the authentication properpties /
methods provided by Laravel before implementing the package.

- [x] Import the `VerifiesUsers` trait (mandatory)
- [  ] Overwrite and customize the redirect path attributes/properties (not
    mandatory)
- [  ] Overwrite and customize the view name for the `getVerificationError()` method
    (not mandatory)
- [x] Create the verification error view according to the defined path (mandatory)
- [  ] Overwrite the contructor (not mandatory)
- [x] Overwrite the `postRegister()`/`register()` method depending on the
    Laravel version you use (mandatory)

```
    ...

    use Jrean\UserVerification\Traits\VerifiesUsers;
    use Jrean\UserVerification\Facades\UserVerification;

    ...

    use VerifiesUsers;

    ...

    /**
     * Create a new authentication controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        // Based on the workflow you want you may update and customize the following lines.

        // Laravel 5.0.*|5.1.*
        $this->middleware('guest', ['except' => ['getLogout']]);

        // Laravel 5.2.*
        $this->middleware('guest', ['except' => ['logout']]);
    }

    ...

    // Laravel 5.0.*|5.1.*

    /**
     * Handle a registration request for the application.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function postRegister(Request $request)
    {
        $validator = $this->validator($request->all());

        if ($validator->fails()) {
            $this->throwValidationException(
                $request, $validator
            );
        }

        $user = $this->create($request->all());

        // Authenticating the user is not mandatory at all.
        //Auth::login($user);

        UserVerification::generate($user);

        UserVerification::send($user, 'My Custom E-mail Subject');

        return redirect($this->redirectPath());
    }

    // Laravel 5.2.*
    /**
     * Handle a registration request for the application.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function register(Request $request)
    {
        $validator = $this->validator($request->all());

        if ($validator->fails()) {
            $this->throwValidationException(
                $request, $validator
            );
        }

        $user = $this->create($request->all());

        // Authenticating the user is not mandatory at all.

        // Laravel <= 5.2.7
        // Auth::login($user);

        // Laravel > 5.2.7
        Auth::guard($this->getGuard())->login($user);

        UserVerification::generate($user);

        UserVerification::send($user, 'My Custom E-mail Subject');

        return redirect($this->redirectPath());
    }
```

Edit the `app\Http\routes.php` file.

- Define two (2) new routes. Routes are customizable.
Don't forget to update the previous listed redirect attributes/properties if you want to
change the pre-defined routes.

```
    Route::get('verification/error', 'Auth\AuthController@getVerificationError');
    Route::get('verification/{token}', 'Auth\AuthController@getVerification');
```

## Contribute

Feel free to comment, contribute and help. 1 PR = 1 feature.

## License

Laravel User Verification is licensed under [The MIT License (MIT)](LICENSE).
