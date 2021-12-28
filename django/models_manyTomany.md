# Models

### Many-To-Many refering to itself, example employee and manager
Via a many to many people I want to be able to track the manager of an employee.
To keep consistency the manager is also part of the employee table as employee.
I track this via a many-to-many people where I have employee, supervisor ( = manager) the role ( = people-manager)
and starting and ending date. 

*Hereby an extract of the model definition :* 

```
class Employee(models.Model):
    uuid = models.UUIDField(db_index=True, default=uuid_lib.uuid4,editable=False)
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30,blank=True)
    email = models.EmailField(blank=True)
    remark = models.TextField(max_length=200, blank=True)
    started_at = models.DateField(default=return_default_startdate)
    ended_at = models.DateField(default=return_default_enddate)
     
    roles = models.ManyToManyField(Role, through="EmployeeRole")
    teams = models.ManyToManyField(Team, through="TeamMember") 
    supervisor = models.ManyToManyField(
        'self',
        through="EmployeeSupervisor",
        through_fields=('employee', 'supervisor')
    )      
    ...
```
And the class related to the many-to-many table *EmployeeSupervisor*
```
class EmployeeSupervisor(models.Model):
    uuid = models.UUIDField(db_index=True, default=uuid_lib.uuid4,editable=False)
    employee = models.ForeignKey(       
        Employee,
        on_delete=CASCADE,
        related_name="employee"             
    )
    supervisor = models.ForeignKey(
        Employee,
        on_delete=CASCADE,
        related_name="supervisor"               
    )
    report_kind = models.ForeignKey(       
        Role,
        on_delete=CASCADE,
        related_name="report_kind",
        blank=True, 
        null=True            
    )
    remark = models.TextField(max_length=200,blank=True)
    ...
```

### Queryset where we include data from another model. 
The dataset was created with employee but we want to add something from an object manager. 
This can be done via `annotate` this will add an extra column to the queryset. In this example it
adds the field *mgr*

```
qs = qs.annotate(
               mgr=Subquery(
                    Manager.objects.filter(                        
                       Q(reportee=OuterRef('id')), 
                       Q(active=True),
                       Q(report_kind__role_name='Team Manager'),   
        
                       Q(started_at__lte=datetime.date(ref_date_year,ref_date_month,ref_date_day)) &
                       Q(ended_at__gte=datetime.date(ref_date_year,ref_date_month,ref_date_day))
                                         
                   ).values('manager__first_name')  
         ))
```
