# Odoo Cheatsheet
[![CC-BY-SA 4.0](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)


This repository exists to document my journey and some struggle i face while working with Odoo. this [gists here](https://gist.github.com/rockavoldy/46350ed3d37662ae9e2cc47ea7fb916a) is the first version of this documentation, i move it to a repository to make it easy to navigate, and when someone wants to contributes (Please contribute!)

## Table of Contents
See top-left icon before **README.md** name, it's a ToC you need!

## Contributing
Feel free to create new Pull Request if you want to add, modify, or give more clarification to some of the things that already in here, or anything that can be useful, really!

Just make sure that, every PR you create, it's better to have a description that explain your modification.

## Useful links and articles
- [Widgets in Odoo](https://www.cybrosys.com/blog/widgets-in-odoo)
- [Widgets in Odoo 13](https://supportuae.wordpress.com/2020/07/13/widgets-in-odoo-13/)
- [Fields in Odoo](https://odoo-development.readthedocs.io/en/latest/dev/py/fields.html)
- [Method decorators in Odoo 13](https://www.cybrosys.com/blog/method-decorators-odoo-13)
- [clear related field with onchange when another field that depends on it is changed](https://learnopenerp.blogspot.com/2016/10/onchange-many2one-filed-in-odoo.html)
- [XML xpath cheatsheet](https://devhints.io/xpath) ***NOT** all xpath here is supported by XML Odoo, please do trial-and-error
- [Change the display name using `name_get` function](https://stackoverflow.com/a/62743976)
- [Change search functionality to filter by other field other than default by overriding `name_search` function](https://stackoverflow.com/a/48146210)
- [Some options for many2one widgets (prevent user to open the link, quick create, or edit and such)](https://stackoverflow.com/a/30590267)
- [click-odoo: help to run python code inside odoo environment (can be useful for testing, or to fetch onchange data that are not stored in DB)](https://github.com/acsone/click-odoo)
- [Remove data that have xml-id using `<delete>` tag](https://www.cybrosys.com/blog/delete-record-from-xml-code-odoo)
- [rename custom module and keep the data](https://gist.github.com/antespi/38978614e522b1a99563)
- [How to using ref by XML ID in the XML domain views](https://www.odoo.com/forum/help-1/odoo-8-how-to-use-ref-on-domain-xml-87862)
- [Odoo installation on Mac M1](https://gist.github.com/rockavoldy/b38b253b15b05f3ba5bd3aed54d82aa4)

## Some PDF reporting thingies
### Create custom button to download report
- When the button is in form view that have the same model with your record, you can easily use type="action" and the name referenced to the template.
    ```xml
    <button name="%(studypermit_form)d" string="Download PDF" type="action" />
    ```
    > [!NOTE]
    > `studypermit_form` is a ir.actions.report template
- When you're in different model view, and want to download record(s) from another model (Ex. you have 1 project, and you want to download all project.task report, and the template is for project.task model).
    ```xml
    <button name="action_download" type="object" string="Download PDF" />
    ```
    and inside that action_download is
    ```py
    def action_download(self):
        template = self.env.ref('project_task_form.studypermit_form')
        
        return template.report_action(self.res_id)
    ```
    > [!NOTE]
    > `project_task_form.studypermit_form` is a template created for model project.task

### View report directly via URL
When you're creating new template for report, instead of downloading everytime you want to check the result, [you can easily view the template by accessing it through URL](https://stackoverflow.com/a/54926043), every modern browser will load it directly in the browser, so it not going to fill up your storage.

URL Template
```
http://<server-address:port>/report/<html or pdf>/<module.xml_id>/<document_id>
```
Example:
```
http://localhost:8069/report/pdf/sale.report_saleorder/38
```
Or the html version
```
http://localhost:8069/report/html/sale.report_saleorder/38
```
> [!IMPORTANT]
> Make sure to check the pdf result too beside of the html result, because some style is not supported or can have different behavior in pdf, like css grid-layout. So, keep tinkering~

### Internationalization report odoo 11
- https://stackoverflow.com/questions/60827887/odoo12-can-i-translate-strings-in-reports
- If you are working on i18n, make sure to not use `--dev` tag when running odoo instance, because [Odoo will directly render the data from the views, and skipping i18n data](https://github.com/odoo/odoo/issues/35553), so translation won't work.

### Report not consistent showing header and footer
- Check your terminal or log, if there is a WARNING from wkhtmltopdf like this
    ```sh
    WARNING <db_name> odoo.addons.base.models.ir_actions_report: wkhtmltopdf: b'Exit with code 1 due to network error: UnknownContentError\n'
    ```
    Then it's mean you have [some resouces that can't be loaded, check again your resources like image, js, css, or font](https://stackoverflow.com/a/57991849/13028862). It's better to put it inside the module at the same server than using CDN, removing 1 point of failure.

## Backup and restore db (not specifically Odoo, but will be used every day)
### Restore DB
1. When it's dumped as gz, use gunzip to extract and pipe directly to psql
    ```sh
    gunzip -c <filename.gz> | psql <dbname>
    ```
2. When it's dumped with pg_dump, use pg_restore and restore with no-owner, so new db will reset the owner, and fix the error
    ```sh
    pg_restore -d <dbname> --role=<role> -O <filename.dump>
    ```
3. When it's dumped as zip through DB Manager Odoo, restore again the DB via DB Manager Odoo
    
    `http://<server-address:port>/web/database/manager`
    > [!NOTE]
    > You can clean-up some data in db (like removing records from ir_mail_server table) by extracting the zip first, open `dump.sql` and remove some lines from it, then zip it again.

### Backup DB
1. When you want to dump it as gunzip, so you can extract and pipe directly to psql
    ```sh
    pg_dump <dbname> | gzip > <filename.gz>
    ```
2. When you want to create backup with pg_restore custom (space friendly like gunzipped)
    ```sh
    pg_dump -Fc <dbname> > <filename.dump>
    OR
    pg_dump -Fc -f <filename.dump> <dbname>
    ```
3. When you want to backup as plain text postgresql query (not space friendly since it's not compressed, but you can directly edit to that query)
    ```sh
    pg_dump -Fp -f <filename.sql> <dbname>
    ```
4. When you want to keep filestore of your db, backup through DB Manager Odoo on
    
    `http://<server-address:port>/web/database/manager`

## Odoo unit test
### Odoo 11
Odoo 11 didn't have a way to run specific unit test, if you run the instance by using tag `--test-enable` only, and update your module, all dependencies modules will run the test too, and it will take times to finish. So, to make sure the test is only run on your module that are intended to upgrade/install, you can add `--stop-after-init`
```sh
python3 odoo-bin -c <conf_file.conf> -d <db_name> --test-enable --stop-after-init -u <module to test>
```
> [!NOTE]
> change `-u` to `-i` if the module haven't been installed.

### Odoo 12 and Odoo 13
On Odoo 12 and 13, There is new way to run specific test using `--test-tags`. To implement it on your unit test, you can find the [documentation here](https://www.odoo.com/documentation/13.0/developer/reference/addons/testing.html#test-selection). and run the test with flag `--test-tags` like below
```sh
python3 odoo-bin -c <conf_file.conf> -d <db_name> --test-enable --test-tags "tag_1,tag_2,tag_3" --stop-after-init -u <module to test>
```
> [!NOTE]
> change `-u` to `-i` if the module haven't been installed.

### Odoo 14 and Odoo 15
And there is another improvement to how flag `--test-tags` works, you can point it directly to the test method or class, so you don't need to set the tags when you want to run specific unit test, but don't want to change the code. And your CLI command will be like this
```sh
python3 odoo-bin -c <conf_file.conf> -d <db_name> --test-enable --test-tags "/<module_name>:<class>.<method>" --stop-after-init -u <module to test>
```
See the format for flag `--test-tags` [in this documentation](https://www.odoo.com/documentation/15.0/developer/reference/cli.html#testing-configuration) and [this invocation documentation](https://www.odoo.com/documentation/15.0/developer/reference/backend/testing.html#invocation).

## Many2many field notation, and how to use them
This notation is useful when you want to manipulate many2many or one2many fields
> [!IMPORTANT]
> Odoo 16 changed this notation with odoo.fields.Command, below notation sometimes still work, but better [to use the new Command for Odoo 16 (and maybe newer Odoo version)](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Command)

- (0, 0, { values }) link to a new record that needs to be created with the given values dictionary
- (1, ID, { values }) update the linked record with id = ID (write values on it)
- (2, ID) remove and delete the linked record with id = ID (calls unlink on ID, that will delete the object completely, and the link to it as well)
- (3, ID) cut the link to the linked record with id = ID (delete the relationship between the two objects but does not delete the target object itself)
- (4, ID) link to existing record with id = ID (adds a relationship)
- (5) unlink all (like using (3,ID) for all linked records)
- (6, 0, [IDs]) replace the list of linked IDs (like using (5) then (4,ID) for each ID in the list of IDs)

[Source](https://stackoverflow.com/a/10612999/13028862)

1. Say, you want to add new group `maintenance.group_user_manager` to your view groups_id. it can be done with id 4 or 6, but if you want to append to existing group that already set somewhere, it must convenient to use id 4. the usage will be
```xml
<field name="groups_id" eval="(4, ref('maintenance.group_user_manager')" />
```
want to use them on python? can do it easily with ORM write (or update, or assignment directly as you would to change the field value)
```python
asset = self.env['account.assets.assets'].browse(50)
asset.write({
    'depreciation_line_ids': (0, 0, {
            'name': '....',
            .....
        })
})
```

1. And say, you are in app Sale, and you have some product_line, you want to remove some line but don't want to remove the product completely, can use `(3, id_of_the_line_you_want_to_remove)`

## Use Odoo with VSCode Debugger
Current version of VSCode (1.64.0 at the time this written) has plenty of features that can be used to improve developer experience, one of this feature is Debugger. With this debugger, you can create a breakpoint and odoo will pause it on that breakpoint, then you can see the call stack that will reach to that breakpoint. So for odoo, we will use [debugpy](https://github.com/microsoft/debugpy). This debugpy can run as a standalone app and something can attach to the listen host and port you already defined with flag `--listen`, but vscode can make it listen directly so it will more simple to use.
Run debugpy from terminal, and wait for anything attached to debugpy
```sh
python3 -m debugpy --wait-for-client --listen 0.0.0.0:5678 odoo-bin <your-usual-odoo-command>
```
Example: 
```sh
python3 -m debugpy --wait-for-client --listen 0.0.0.0:5678 odoo-bin -c odoo.conf -d odoo13
```

For this one, i have an assumption that you already have your configuration to run and develop your odoo locally, so we just need to migrate that command or configuration to vscode launch.json
1. Install debugpy first with pip
    ```sh
    pip3 install debugpy
    ````
2. Then create directory `.vscode` inside your workspace and a new file called `launch.json` inside that `.vscode` directory.
3. See below json script, this is the example to run odoo with debugpy on the `launch.json` file. Personalize it with your configuration to the args section. Usually, i will run odoo with command
    ```sh
    <cd to the workspace directory>
    python3 odoo-bin -c odoo.conf -d odoo13
    ```
    So, the launch.json can be
    ```json
      {
        "version": "0.2.0",
        "configurations": [
          {
            "name": "Odoo - Start With Debugger",
            "type": "python",
            "python": "<absolute-path-to-your-python-bin>",
            "request": "launch",
            "module": "debugpy",
            "stopOnEntry": false,
            "args": [
              "--wait-for-client",
              "${workspaceRoot}/odoo-bin",
              "-c",
              "${workspaceRoot}/odoo.conf",
              "-d",
              "odoo13"
            ],
            "cwd": "${workspaceRoot}"
          },
          {
            "name": "Python - Attach debugger",
            "type": "python",
            "request": "attach",
            "port": 5678,
            "host": "localhost"
          }
        ]
      }
    ```
    change args section with your way to run odoo. in my launch.json above the first line is flag from debugpy, then it will run the odoo by specify where the app is, and followed by your odoo configuration. You can find another flag for [debugpy in the project repository](https://github.com/microsoft/debugpy). 
    
    As for the `${workspaceRoot}`, this is the directory you open the workpace on that vscode, say i open this workspace inside directory `/opt/odoo`, then the `${workspaceRoot}` will be `/opt/odoo`
4. And now you can run it directly from "Run and Debug" menu, usually this is below the "Search" icon on the left panel
5. Now you can use the breakpoint feature too, and it will show you the call stack to reach that function from the first method called until paused by the breakpoint.

## Minimal odoo.conf
When you run odoo by running it only using argument, odoo will automatically create default `.odoorc` file (or something like that) that will be used as default odoo configuration. This `.odoorc` here sometimes have higher priority than your `odoo.conf` or the arguments, so there is a chance when you have 2 separate custom-addons directory, and you are trying to exclude 1 directory from `addons_path` parameter, the path can still be included. So, it's better to use separate `odoo.conf` for every project when you're developing custom module in the same machine.
```sh
[options]
db_name = False
db_user = odoo
db_password = False
addons_path = addons, odoo/addons, enterprise, custom-addons
```
Above `odoo.conf` file is very minimal. You can put it at the same level as `odoo-bin`, then you can run it with
```sh
python3 odoo-bin -c odoo.conf -d <db-name>
```
i didn't put `db_name` inside the conf to make it easier when i need to switch DB on the same project. So, i can easily change db for testing and development.