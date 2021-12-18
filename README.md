<h1>Laravel/sanctum with API authentication</h1>
<h2>From scratch</h2>
    <ul>
        <li>composer create-project laravel/laravel laravel-project</li>
        <li>composer require laravel/sanctum</li>
        <li>php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"</li>
        <li>in kernel.php file:<br/>
            <pre>
            'api' => [
                \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
                'throttle:api',
                \Illuminate\Routing\Middleware\SubstituteBindings::class,
            ],
            </pre>
        </li>
        <li>In Model User, make sure user use HasApiTokens<br/>
            <pre>
            class User extends Authenticatable
            {
                use <strong>HasApiTokens</strong>, HasFactory, Notifiable;
                ...
            </pre>
        </li>
        <li>Create routes in api.php. Don't forget to add AuthenticationController.<br/>
            <pre>
                use Illuminate\Http\Request;
                use Illuminate\Support\Facades\Route;
                use App\Http\Controllers\AuthenticationController;
                ...
                //register new user
                Route::post('/create-account', [AuthenticationController::class, 'createAccount']);
                //login user
                Route::post('/signin', [AuthenticationController::class, 'signin']);
                //using middleware
                Route::group(['middleware' => ['auth:sanctum']], function () {
                    Route::get('/profile', function(Request $request) {
                        return auth()->user();
                    });
                    Route::post('/sign-out', [AuthenticationController::class, 'logout']);
                });
            </pre>
        </li>
        <li>php artisan make:controller AuthenticationController</li>
        <li>Create signin, signout and createAccount functions. Don't forget to add User model and Auth facade<br/>
            <pre>
                namespace App\Http\Controllers;
                use Illuminate\Http\Request;
                use Illuminate\Support\Facades\Auth;
                use App\Models\User;
                ...
                class AuthenticationController extends Controller
                {
                    //this method adds new users
                    public function createAccount(Request $request)
                    {
                        $attr = $request->validate([
                            'name' => 'required|string|max:255',
                            'email' => 'required|string|email|unique:users,email',
                            'password' => 'required|string|min:6|confirmed'
                        ]);
                        $user = User::create([
                            'name' => $attr['name'],
                            'password' => bcrypt($attr['password']),
                            'email' => $attr['email']
                        ]);
                        return $this->success([
                            'token' => $user->createToken('tokens')->plainTextToken
                        ]);
                    }
                    //use this method to signin users
                    public function signin(Request $request)
                    {
                        $attr = $request->validate([
                            'email' => 'required|string|email|',
                            'password' => 'required|string|min:6'
                        ]);
                        if (!Auth::attempt($attr)) {
                            return $this->error('Credentials not match', 401);
                        }
                        return $this->success([
                            'token' => auth()->user()->createToken('API Token')->plainTextToken
                        ]);
                    }
                    // this method signs out users by removing tokens
                    public function signout()
                    {
                        auth()->user()->tokens()->delete();
                        return [
                            'message' => 'Tokens Revoked'
                        ];
                    }
                }
            </pre>
        </li>
        <li>In Controller.php, add success and error functions<br/>
            <pre>
                namespace App\Http\Controllers;
                use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
                use Illuminate\Foundation\Bus\DispatchesJobs;
                use Illuminate\Foundation\Validation\ValidatesRequests;
                use Illuminate\Routing\Controller as BaseController;
                ...
                class Controller extends BaseController
                {
                    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;
                    public function success($result, $msg='')
                    {
                        $res = [
                            'success' => true,
                            'data'    => $result,
                            'message' => $msg,
                        ];
                        return response()->json($res, 200);
                    }
                    public function error($error, $errorMsg = [], $code = 404)
                    {
                        $res = [
                            'success' => false,
                            'message' => $error,
                        ];
                        if(!empty($errorMsg)){
                            $res['data'] = $errorMsg;
                        }
                        return response()->json($res, $code);
                    }
                }
            </pre>
        </li>
    </ul>
</h2>
<h2>If the project is cloned</h2>
<ul>
    <li>composer install</li>
    <li>cp .env.example .env</li>
    <li>Edit .env</li>
    <li>php artisan key:generate</li>
    <li>php artisan migrate</li>
</ul>
