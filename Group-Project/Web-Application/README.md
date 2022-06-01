#Web Application
## The Code
See our [GitHub Repositories](https://github.com/s3gp-foodies) for the full code

## What I Worked On
Originally I had planned to mainly work on the backend - .NET Core - for this project since others had indicated previous experience with Vue. This pretty quickly turned out to be minimal and thus I ended up working on both sides of the project. I primarily set up the major components and systems, helped others and refactored code that others had written to improve its functionality and/or readability.

In this document I will be highlighting some of the things I implemented in the project.

## [Back End](https://github.com/s3gp-foodies/restaurant-backend)
In the backend I set up a .NET Core 6 API using the Controller-Repository pattern. The Controllers make use of an injected Unit-of-Work pattern that provides access to the repositories while making sure a single data context is shared between them.
```cs
public class UnitOfWork : IUnitOfWork
{
    private readonly DataContext _context;
    private readonly IMapper _mapper;

    public UnitOfWork(DataContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }

    public ICategoryRepository CategoryRepository => new CategoryRepository(_context, _mapper);
    public IMenuRepository MenuRepository => new MenuRepository(_context, _mapper);
    public IOrderRepository OrderRepository => new OrderRepository(_context, _mapper);
    public ISessionRepository SessionRepository => new SessionRepository(_context, _mapper);

    public async Task<bool> Complete()
    {
        return await _context.SaveChangesAsync() > 0;
    }

    public bool HasChanges()
    {
        return _context.ChangeTracker.HasChanges();
    }
}
```
User authentication is done via EF Core Identity in combination with a JWT Token Service
```cs
public class TokenService : ITokenService
{
    private readonly UserManager<AppUser> _userManager;
    private readonly SymmetricSecurityKey _key;

    public TokenService(IConfiguration config, UserManager<AppUser> userManager)
    {
        _userManager = userManager;
        _key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(config["TokenKey"]));
    }

    public async Task<string> CreateToken(AppUser user)
    {
        var claims = new List<Claim>
        {
            new(JwtRegisteredClaimNames.UniqueName, user.UserName),
            new(JwtRegisteredClaimNames.NameId, user.Id.ToString())
        };

        var roles = await _userManager.GetRolesAsync(user);

        claims.AddRange(roles.Select(role => new Claim(ClaimTypes.Role, role)));

        var creds = new SigningCredentials(_key, SecurityAlgorithms.HmacSha512Signature);

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.Now.AddDays(7),
            SigningCredentials = creds
        };

        var tokenHandler = new JwtSecurityTokenHandler();

        var token = tokenHandler.CreateToken(tokenDescriptor);

        return tokenHandler.WriteToken(token);
    }
  }

```
I also made use of middleware to catch errors in http requests and filter the amount of information that is returned with the error based on whether the application is running in dev environment or not, including a custom API Exception.

## Front End
For the frontend we used Vue 3. This was a mistake as many tools such as design frameworks like material ui and bootstrap don't yet have proper support for Vue 3.

By the end of the first sprint it was clear that our FE team did not have much experience working with Vue or another component based Javascript framework. In sprint 2 I refactored a lot of the existing FE code to be component based (See commits on 19-04-22) after which I explained how components work to the team.

I used dependency injection to make services available to components and to make the websocket available to our other services. Although the solution works it is a bit of an anti-pattern as it is not compliant with Vue's design philosophy due to how it handles data storage.
```js
class AccountService extends SocketConsumer {
    // Add websocket functions in this constructor
    Init(){
    }

    Login(user: User) {
        return axios
            .post(API_URL + "login", user)
            .then(async (response) => {
                localStorage.userId = response.data.id;
                localStorage.userName = response.data.userName;
                localStorage.token = response.data.token;
                await this._socketService?.Connect()
                return true
            })
            .catch((error) => {
                console.log(error);
                return false
            });
    }
}
```
```js
@Options({
  components: {SocketContainer},
  provide: {
    accountService: new AccountService(),
    menuService: new MenuService(),
    orderService: new OrderService(),
  }
})

export default class App extends Vue {
}
```
```js
export default {
  name: "MenuList",
  inject:['menuService'],
  ...
  created() {
    this.menuService.Load().then(() => {
      this.categories = this.menuService.GetCategories();
      this.categories.forEach(cat => {
        this.menuPerCategory[cat.id] = (this.menuService.GetItemsInCategory(cat))
      })
      this.isLoading = false;
    })
  }
```
A solution to some of the issues we ran into with the services I implemented is Vuex - a centralized storage for a Vue webapp. In sprint 4 I ended up adding Vuex to our application and moving some of the data over to there - including the menu from the example above.

```js
export const store = createStore({
    state: {
        menu: new Menu([]),
        categories: []
    },
    mutations: {
        AddToMenu(state, product: Product) {
            state.menu.products.push(product)
        },
        AddToCategories(state, {id, name}) {
            const cat = new Category(id, name)
            state.categories.push(cat)
        }
    },
    getters: {
        GetProductById: (state) => (id: number) => {
            return state.menu.products.find(p => p.id == id);
        },
        GetItemsInCategory: (state) => (category: Category) => {
            return state.menu.products.filter(item => item.category.id === category.id)
        },
        GetCategory: (state) => (categoryId: number) => {
            return state.categories.find(cat => cat.id === categoryId)
        },
        // This is called way too many times but that's a vue 3.0 issue
        // caching doesn't work, will be fixed in 3.1
        GetCategorizedMenu(state) {
            const result = {}
            state.categories.forEach(category => {
                result[category.id] = state.menu.products.filter(item => item.category.id === category.id)
            })
            return result
        }
    }
})
```

```js
export default {
  name: "MenuList",
  ...
  computed: {
    categories() {
      return store.state.categories
    },
    menuPerCategory() {
      return store.getters.GetCategorizedMenu
    }
  },
  inject: ['menuService'],

  created() {
    this.menuService.Load().then(() => {
      this.isLoading = false;
    })
  }
}
```
I added error handling to the website by adding a Toast handler and using an axios interceptor to show errors with a toast popup.
```js
export default function axiosRequestInterceptors() {
    const toast = useToast();
    axios.interceptors.response.use(function (response) {
        // Any status code that lie within the range of 2xx cause this function to trigger
        // Do something with response data
        return response;
    }, function (error) {
        // Any status codes that falls outside the range of 2xx cause this function to trigger
        // Do something with response error
        toast.error(error.message)
        return Promise.reject(error);
    });

}
```
