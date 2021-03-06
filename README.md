# Skiff

Python Wrapper for DigitalOcean's v2 API

<img src="https://raw.githubusercontent.com/Shrugs/skiff/master/media/SkiffLogo.png" width=200>

This project is not actively maintained in sync with v2 API changes; if something changes (and therefore breaks the library), open an issue and I will fix it as soon as possible.

## Installation

```python
    pip install skiff
```

## Basic Usage
```python
    import skiff
    s = skiff.rig("my_token")
    droplets = s.Droplet.all()
    #>>> [<cond.in (#267357) nyc1 - Ubuntu 12.10 x64 - 512mb>,
    #>>> <hey.github (#2012972) nyc1 - Ubuntu 13.10 x32 - 512mb>,
    #>>> <hello.world (#2012974) nyc1 - Ubuntu 13.10 x32 - 512mb>]
```

## API

### General Notes

Creation calls can be made with keyword args or a single dictionary argument.

Calls to `s.<resource>.all()` can also be passed query parameters. The v2 API currently responds to `per_page`. For example, `s.Image.all(per_page=200)` will return all of the images up to the 200th (200 is max). By default, `s.<resource>.all()` will page through all responses. Pass `page=False` to stop skiff from automatically retriving all of the possible values.

Each object has a property `_json`, which contains the json response as a dictionary. If you make local changes to the skiff objects (via `update()` or similar), be sure to refresh the local json with `my_object = my_object.reload()`.

### Droplets

#### Create
```python
    my_droplet = s.Droplet.create(name='hello.world',
                                  region='nyc1',
                                  size='512mb',
                                  image=5141286)
    #>>> <hello.world (#2012974) nyc1 - Ubuntu 14.04 x64 - 512mb>
    # wait until droplet is created
    my_droplet.wait_till_done()
    # refresh network information
    my_droplet = my_droplet.refresh()
```
Alternatively, you can pass a dictionary containing those values.

#### Get
```python
    # Get droplet by ID
    my_droplet = s.Droplet.get(id)
    # Get droplet by Name (not intelligent)
    my_droplet = s.Droplet.get('hello.world')
```

#### Destroy
```python
    my_droplet.destroy()
```

#### Rename
```python
    my_droplet.rename('new.name')
```

#### Reboot
```python
    my_droplet.reboot()
    my_droplet.restart()
```

#### Shutdown
```python
    my_droplet.shutdown()
```

#### Power Off
```python
    my_droplet.power_off()
```

#### Power On
```python
    my_droplet.power_on()
```

#### Power Cycle
```python
    my_droplet.power_cycle()
```

#### Resize
```python
    # Get size via search
    some_size = s.Size.get('512')
    #>>> <512mb>
    my_droplet.resize(some_size)
```

Alternatively, simply pass in the string `'512mb'`.

#### Rebuild
```python
    ubuntu_image = s.Image.get('Ubuntu 13.10')
    my_droplet.rebuild(ubuntu_image)
```

#### Restore
```python
    # Default to current Image
    my_droplet.restore()
    # Specify Image
    ubuntu_image = s.Image.get('Ubuntu 13.10')
    my_droplet.restore(ubuntu_image)
```

#### Password Reset
```python
    my_droplet.password_reset()
    my_droplet.reset_password()
```

#### Change Kernel
```python
    new_kernel = my_droplet.kernels()[10]
    my_droplet.change_kernel(new_kernel)
```

Alternatively, simply pass the kernel's ID.

#### Enable IPv6
```python
    my_droplet.enable_ipv6()
```

#### Create Snapshot
_Note: Droplet must be powered off before you perform this._

```python
    my_droplet.snapshot()
```
#### Disable Backups
```python
    my_droplet.disable_backups()
```

#### Enable Private Networking
```python
    my_droplet.enable_private_networking()
```

#### Get Droplet Snapshots
```python
    my_droplet.snapshots()
```

#### Get Droplet Backups
```python
    my_droplet.backups()
```

#### Get Droplet Actions
```python
    my_droplet.actions()
```

#### Get Droplet Kernels
```python
    my_droplet.kernels()
```

### Droplet Helper Methods

#### refresh/reload
When the droplet's status changes remotely, call this function to manually update your local representation.

**This is particularly useful after creating a new droplet because the network info is blank at request time.**

```python
    my_droplet = my_droplet.refresh()
    my_droplet = my_droplet.reload()
```

#### has_action_in_progress
Returns a boolean regarding whether or not the droplet is processing an action.

```python
    my_droplet.has_action_in_progress()
    #>>> False
```

#### wait_till_done
Blocks until the droplet has no more pending actions.

```python    
    # waits until the droplet is ready to use, polling every 5 seconds
    my_droplet.wait_till_done(5)
```

### Actions

#### Get All Actions
Returns all actions for a token.

```python
    s.Action.all()
```

#### Get Action by ID
```python
    action_id = 28012139
    s.Action.get(action_id)
    #>>> <destroy (#28012139) completed>
```
### Domains

#### Get All Domains
```python
    s.Domain.all()
    #>>> [<blog.cond.in>,
    #>>> <matt.cond.in>,
    #>>> <example.com>,
    #>>> <example2.com>]
```

#### Create Domain
```python
    # easy, defaulting to fist ipv4 network's public ip
    my_domain = my_droplet.create_domain('example.com')

    # or more manually
    my_domain = s.Domain.create(name='example.com', ip_address=my_droplet.v4[0].ip_address)
```

#### Get Specific Domain
```python
    my_domain = s.Domain.get('example.com')
    my_domain = s.Domain.get(domain_id)
```

#### Delete/Destroy Domain
These are aliases for the same method.

```python
    my_domain.delete()
    my_domain.destroy()
```

### Domain Records

#### Get Domain Records
```python
    my_domain.records()
    #>>>[<example.com - A (#348736) @ -> 123.456.789.123>,
    #>>> <example.com - CNAME (#348740) www -> @>,
    #>>> <example.com - NS (#348737)  -> NS1.DIGITALOCEAN.COM.>,
    #>>> <example.com - NS (#348738)  -> NS2.DIGITALOCEAN.COM.>,
    #>>> <example.com - NS (#348739)  -> NS3.DIGITALOCEAN.COM.>]
```

#### Create Domain Records
```python
    my_domain.create_record(type='CNAME', name='www', data='@')
```

See the [DigitalOcean v2 API Docs](https://developers.digitalocean.com/v2/#create-a-new-domain-record) for more options.

#### Get Domain Record by ID
```python
    my_record_id = 1234
    my_record = my_domain.get_record(my_record_id)
```

#### Update Domain Record
```python
    my_record = my_record.update('new_name')
```

See the [DigitalOcean v2 API Docs](https://developers.digitalocean.com/v2/#update-a-domain-record) for information on what `new_name` should be.

#### Delete/Destroy Domain Record
These are aliases for the same method.

```python
    my_record.delete()
    my_record.destroy()
```

### Images

#### List All Images
```python
    s.Image.all(per_page=100)
    #>>> [<CentOS 5.8 x64 (#1601) CentOS>,
    #>>> <CentOS 5.8 x32 (#1602) CentOS>,
    #>>> ...........
    #>>> <Ghost 0.5 on Ubuntu 14.04 (#5610254) Ubuntu>,
    #>>> <Redmine on Ubuntu 14.04 (#4869208) Ubuntu>,
    #>>> <WordPress on Ubuntu 14.04 (#4991187) Ubuntu>]
```

#### Get Image by ID, Slug, or Search
```python
    # Get by ID
    my_image_id = 3101580
    my_image = s.Image.get(my_image_id)
    # Or by slug
    ubuntu_slug = 'ubuntu1404'
    ubuntu_image = s.Image.get(ubuntu_slug)
    # Or by search (not very intelligent; useful for REPL use)
    ubuntu_image = s.Image.get('Ubuntu 13.10')
```

#### Delete/Destroy an Image
These are aliases for the same method.

```python
    my_image.delete()
    my_image.destroy()
```

#### Update an Image
```python
    name = 'my_new_image'
    my_image = my_image.update(name)
```

#### Transfer Image
```python
    new_region = s.Region.get('nyc1')
    my_image.transfer(new_region)
```

Alternatively, simply pass the string 'nyc1'.

#### Get Image Actions
```python
    my_image.actions()
```

#### Get Specific Image Action by ID
```python
    action_id = 1234
    my_image.get_action(action_id)
```

### Keys

#### Get All Keys
```python
    s.Key.all()
```

#### Get Specific Key by ID, Name or, Fingerprint
```python
    # ID
    my_key = s.Key.get(1234)
    # Name
    my_key = s.Key.get('my public key')
    # Fingerprint
    my_key = s.Key.get('my:fi:ng:er:pr:in:t!')
```

#### Create New Key
```python
    with open('~/.ssh/id_rsa.pub', 'r') as f:
        pub_key = f.read()
    my_key = s.Key.create(name='my public key', public_key=pub_key)
```

#### Update Key
```python
    my_key = my_key.update('new public key name')
```

#### Delete/Destroy Key
These are aliases for the same method.

```python
    my_key.delete()
    my_key.destroy()
```

### Regions

#### List All Regions
```python
    s.Region.all()
    #>>> [<New York 1 (nyc1)>,
    #>>> <San Francisco 1 (sfo1)>,
    #>>> <New York 2 (nyc2)>,
    #>>> <Amsterdam 2 (ams2)>,
    #>>> <Singapore 1 (sgp1)>]
```

#### Get Specific Region by Slug
There's probably not much benefit in getting a SkiffRegion instance rather than just passing the region slug string as a parameter.

```python
    nyc1_region = s.Region.get('nyc1')
```

### Sizes

#### Get all Sizes
```python
    s.Size.all()
    #>>> [<512mb>, <1gb>, <2gb>, <4gb>, <8gb>, <16gb>, <32gb>, <48gb>, <64gb>]
```

### Get Specific Size
```python
    # search, not intelligent
    small_size = s.Size.get('512')
```

## Contributors

- [lost-theory](https://github.com/lost-theory)
- [cstrutton](https://github.com/cstrutton)
- [fhats](https://github.com/fhats)

## TODO

Suggestions accepted via Issues and Pull-Requests :)

## Testing

run py.test

Note: Takes a while.

    py.test

## License 

MIT