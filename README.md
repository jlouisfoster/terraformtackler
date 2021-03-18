# Place to store all your tips & tricks for Terraform

### Azure Devops Process:

1. Test by running pipeline yaml for code errors directly in azure devops prior to creating pull request

2. Create pull request once tests pass and once approved - and merged to master, it will auto deploy with service principal to azure where your scaleset will be created.



### Terraform Variables ELI5

```
anything in a "variable" block is a variable and it's scope is that it can be accessed by any file ending with ".tf" in the same directory. The "variable" block is where you say "hey Terraform, there's a thing that I want to call 'foo' and if it isn't the type that I say it is or if it's missing, throw an error so I know that I done fucked up."

Now what's the value of a variable? If it's declared in a module, then when you declare a module you'll pass "foo = <some value" for everything that's inside of a "variable" block in a .tf file that's in the same directory that your "source = /path/to/module" is. What if it's the root module? You don't declare those, you just run "terraform plan" or something. Where do those values come from? Well, you could pass them in on the command line or declare an environment variable that has a similar name, but that kinda sucks to type every time (unless it's something like a password, in which case you want to type it every time). For that, you can define the variables in a .tfvars file by saying `foo = "value"` or if it's more complicated, `foo = { derp = "1", "baz" = 2 }` or something like that. When terraform runs, you can tell it "here's a file that ends in .tfvars" and it'll look there for the values it should use for a variable. Don't like telling terraform what file to check every time? No problem, name the file <whatever>.auto.tfvars and Terraform will always assume that the values you want to pass in to your variables are in that file.

Now, what if I have some crazy conditional that I'm copy-pasting everywhere on a bunch of resources? I don't want to copy-paste stuff, so I can instead declare a variable in the "locals" section and then reference it with "local.variablename" whenever I want to use it, but only in .tf files that are in the same directory. This is handy if you have a complicated condition around a "for_each" or "count" for a bunch of the same resources. It's also useful if you need to extract specific data from a map. For example, if I have a variable that's a map of usernames with email, value, and a list of groups, I could use a variable in the locals section that just grabs the usernames and pass that to a resource to create users. I could create another one that grabs only the usernames of everyone who has "admin" in their groups list, which would be pretty handy to pass to a resource that adds people to a group called "admin" rather than have to re-declare the same data a bunch of times.

There are lots of other things you can do too, but it's also important to know where you *can't* use variables:

- module source="must/be/hard-coded/path"
- provider_alias = aws.mustbehardcoded (same with a providers block to a module)
- terraform state backend configuration

- resource "novariableshere"
Hope that helps. Feel free to ask other questions if you need more information and I'll try to get back to you.
Source: https://www.reddit.com/r/Terraform/comments/jphmpl/eli5_terraform_variables/
```

### Terraform Resources:

e.g
```
resource "azurerm_resource_group" "cloudagent" {
    name = module.namming.ressourceGroupName
    location = local.location   
    tags = var.resource_tags
```

Q: What does the cloudagent mean here? I understand that this uses the azure resource block in terraform to obtain the resource group but is "cloudagent" the name you are giving your newly created resource group or is just a placeholder name to be referenced later? because I see name = module.namm - which I assume pulls the actual name of the resource group that module is using..?

A: [12:40 PM] Vincent Prince
    It's the name of the terraform ressources itself
​[12:40 PM] Vincent Prince
    it need to be unique within the module
​[12:41 PM] Vincent Prince
    module to module communication, this need to be transfered in an  "output"
​[12:41 PM] Vincent Prince
    but the terraform name and the azure ressource name does not need to match (however it is strongly suggested that they avec a certain link)
​[12:42 PM] Vincent Prince
    ex: in the screenshot you sent met : azurerm_ressource_group.cloudagent will be namme CAE-ENG-LABS-STAGE-CloudAgent
​[12:42 PM] Vincent Prince
    or something like that

### Terraform Module Access

Q: [1:18 PM] James Louis-Foster
    and when you call module.naming.ressourceGroupName
​[1:19 PM] James Louis-Foster
    it will ping azure during deployment to get the right name or it uses a variable inside terraform that you've set?

A: [1:25 PM] Vincent Prince
    yes it the root main.tf file we specify a backen
    
​[1:26 PM] Vincent Prince
    it translates the Terraform ressource name to an Azure ID



