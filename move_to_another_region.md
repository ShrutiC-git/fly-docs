<!-- omit in toc -->
# Move Fly App from One Region to Another

You may want to move your [Fly App](https://fly.io/docs/reference/apps/) to a region that is closest to your users. With the availability of many [Fly Regions](https://fly.io/docs/reference/regions/), you can easily move your Fly App from the current region to a region that is most appropriate for your needs. 

This document describes how you can move your Fly App completely from one region to another using [flyctl](https://fly.io/docs/flyctl/), the Fly.io CLI.  

- [Prerequisites](#prerequisites)
- [Steps to Move](#steps-to-move)
  - [Step 1: Move App-Resources to the New Region](#step-1-move-app-resources-to-the-new-region)
    - [Move Resources Using `fly scale count`](#move-resources-using-fly-scale-count)
    - [Move Resources Using `fly machine clone`](#move-resources-using-fly-machine-clone)
  - [Step 2: Remove Machines in the Old Region](#step-2-remove-machines-in-the-old-region)
    - [Remove Machines Using `fly scale count 0`:](#remove-machines-using-fly-scale-count-0)
    - [Remove Machines Using `fly machine destroy`:](#remove-machines-using-fly-machine-destroy)
  - [Step 3: Remove Volumes in the Old Region:](#step-3-remove-volumes-in-the-old-region)


## Prerequisites
All you need to move your App from one region to another is: [flyctl](https://fly.io/docs/flyctl/), and of course your App! That's it!

## Steps to Move 
Moving your Fly App from an old region to a new region is a simple, three-step process, which are pretty self-explanatory: 
1. Move Your [App Resources (Machines and Volumes) to the New Region](#step-1-move-app-resources-to-the-new-region)
2. [Remove Machines in the Old Region](#step-2-remove-machines-in-the-old-region)
3. [Remove Volumes in the Old Region](#step-3-remove-volumes-in-the-old-region)

> Note: The steps in this guide are for moving an App with one Machine and one Attached Volume. 
 

### Step 1: Move App-Resources to the New Region

To move your Machine and Volume resources to the new region, you can either [move using fly scale count](#move-resources-using-fly-scale-count) or [move using fly machine clone](#move-resources-using-fly-machine-clone).

>Note: Since for V2 Fly Apps, all Fly Machines are managed together, either [`fly scale count`](https://fly.io/docs/flyctl/scale-count/) or [`fly machine clone`](https://fly.io/docs/flyctl/machine-clone/) can be used for moving your Machines to another region.  
The `fly scale count` command is more convenient for bulk-moving Machines while `fly machine clone` moves one Machine at a time. 

#### Move Resources Using [`fly scale count`](https://fly.io/docs/flyctl/scale-count/)

`fly scale count` will create as many Fly Machines in the region of your choice as you specify. 

```console
$ fly scale count <number_of_machines_in_the_new_region> --region <new_region_code>
```

**Example:** If you want to move to the `syd` region and want 1 instance of your App in that region, you will need to run:

```console
$ fly scale count 1 --region syd
```

`fly scale count` automatically takes care of any Volumes your App has when it scales Machines. `fly scale count` either creates new Volumes for newly created Fly Machines or it attaches [unattached Volumes](https://fly.io/docs/volumes/overview/#volume-attachment) to the new Machines. There will be as many Fly Volumes as there are Fly Machines. Read more about [how Fly Volumes work](https://fly.io/docs/volumes/overview/). 

**Example:** if the new region has no unattached Volume, `fly scale count 1 --region <region>` will create a new Volume and attach the Volume to the newly created Fly Machine.  If however, the region has an unattached Volume, `fly scale count 1 --region <region>` will attach the unattached Volumes to the newly created Fly Machine. You can change this behavior using the [`--with-new-volumes`](https://fly.io/docs/flyctl/scale-count/#options) flag.

> Note: The `fly scale count` command creates new empty Volumes, or attaches existing Volumes, and does not copy or move any data between Volumes. To create a one-time copy, you can use [`fly volumes fork`](https://fly.io/docs/flyctl/volumes-fork/) or use alternatives like [LiteFS](https://fly.io/docs/litefs/) to sync the new Volumes. 

#### Move Resources Using [`fly machine clone`](https://fly.io/docs/flyctl/machine-clone)

Unlike `fly scale count`, `fly machine clone` creates one new Machine, at a time, which is a copy of the Machine you specify, in the region you specify. 

```console
$ fly machine clone <machine_id_of_the_old_machine> --region <new_region_code>
```
**Example:** if you want to move to the `syd` region, you will need to run:

```console
$ fly machine clone <machine_id> --region syd
```
 
Similar to `fly scale count`, `fly machine clone` automatically takes care of any Volumes your App has when you clone your App to another region. However, unlike `fly scale count`, `fly machine clone` does not use unattached volume in the new region. `fly machine clone` will create a **new empty Volume** and attach it to the new Machine.

### Step 2: Remove Machines in the Old Region
Once you have your App running in the new region, you can [remove Machines using fly scale count 0](#remove-machines-using-fly-scale-count-0) or [remove Machines using fly machine destroy](#remove-machines-using-fly-machine-destroy).

#### Remove Machines Using [`fly scale count 0`](https://fly.io/docs/apps/scale-count/#scale-to-zero-and-back-up):

```console
$ fly scale count 0 --region <old_region>
```
`fly scale count 0` scales to zero Machines in the old region. 

> Note: The `fly scale count 0` will work only for Machines launched using Fly Launch.

#### Remove Machines Using [`fly machine destroy`](https://fly.io/docs/flyctl/machine-destroy):
You can also use `fly machine destroy` to remove Machines in the old region, giving you more control over the order in which Machines are removed.

```console
$ fly machine destroy <machine_ID> --force
```
The above command will destroy the specified Machine. The `--force` flag is required for any Machine currently in the `started` state. You can also, alternatively, stop a Machine first using `fly machine stop <machine_ID>` and then destroy it.


### Step 3: Remove Volumes in the Old Region:
The `fly scale count 0` and the `fly machine destroy` commands only remove Machines, they do not destroy any attached Volumes. When Machines are either scaled to zero or destroyed, the Volumes in that region become unattached. The [`fly volumes destroy`](https://fly.io/docs/flyctl/volumes-destroy/) command destroys unattached Volumes. 

To destroy an unattached Volume in the old region, run:

```console
$ fly volumes destroy <volume_ID>
```

The above command will destroy the specified unattached Volume, along with **all its data**.
