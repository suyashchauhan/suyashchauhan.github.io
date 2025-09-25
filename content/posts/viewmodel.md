+++
date = '2025-05-06T22:24:17+05:30'
title = 'Viewmodel'
+++

How ViewModels are retained on Config Changes?

### Basic Terms

- ViewModelProvider

![alt text](/images/viewModel.png)

- ViewModelStore 
> storage for viewmodel

![alt text](/images/viewModel-2.png)


- ViewModelStoreOwner
> interface which contains storage object and is implemented by ComponentActivity etc.

![alt text](/images/viewModel-5.png)
![alt text](/images/viewModel-6.png)





### Attaching ViewModel to activity

MainActivity.kt
```kt{linenos=inline}
class MainActivity : AppCompatActivity() {
    @RequiresApi(Build.VERSION_CODES.P)
    private lateinit var viewModel: MyViewModel;

    @RequiresApi(Build.VERSION_CODES.P)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        
        viewModel = ViewModelProvider(this)[MyViewModel::class.java]
       
    }
}![alt text](image.png)
```


**Q: Aren't we getting new viewmodel instance everytime as when configuration is changed onCreate is invoked again ?**  
No

Let me walk you through code
![alt text](/images/viewModel-4.png)

See how *ViewModelStore* is returned on from NonConfigurationInstance if not we create a new instance