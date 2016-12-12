# Taskomatic API

## Satellite API


namespace: taskomatic

 * List<TaskoBunch> listSatBunches()
  - lists all available satellite bunches
 * Date scheduleSatBunch(String bunchName, String jobLabel, Date startTime, Date endTime, String cronExpression, Map params) throws NoSuchBunchTaskException, InvalidParamException
  - start scheduling a satellite bunch
 * Date scheduleSatBunch(String bunchName, String jobLabel, String cronExpression, Map params) throws NoSuchBunchTaskException, InvalidParamException
  - start scheduling a satellite bunch
 * Integer unscheduleSatBunch(String jobLabel) throws InvalidParamException
  - stop scheduling a satellite bunch
 * Date scheduleSingleSatBunchRun(String bunchName, Map params, Date start) throws NoSuchBunchTaskException, InvalidParamException
  - schedule a one time satellite bunch
 * Date scheduleSingleSatBunchRun(String bunchName, Map params) throws NoSuchBunchTaskException, InvalidParamException
  - schedule a one time satellite bunch asap
 * List<TaskoSchedule> listAllSatSchedules()
  - lists all satellite schedules
 * List<TaskoSchedule> listActiveSatSchedules()
  - lists all active satellite schedules
 * List<TaskoSchedule> listActiveSatSchedulesByBunch(String bunchName) throws NoSuchBunchTaskException
  - lists all active satellite schedules associated with given bunch
 * List<TaskoRun> listScheduleSatRuns(Integer scheduleId)
  - lists all satellite runs of a give schedule
 * String getSatRunStdOutputLog(Integer runId, Integer nBytes) throws InvalidParamException
  - get last specified number of bytes of the satellite run std output log, whole log is returned if nBytes is negative
 * String getSatRunStdErrorLog(Integer runId, Integer nBytes) throws InvalidParamException
  - get last specified number of bytes of the satellite run std error log, whole log is returned if nBytes is negative
 * List<TaskoSchedule> reinitializeAllSchedulesFromNow()
  - reinitialize all schedules
  - meant to be called, when taskomatic has to be reinitialized (f.e. because of time shift)

*Note:* To call the methods over tomcat, make sure you prepend sessionKey as the first parameter.
## Org API

namespace: taskomatic.org

 * int one(Integer orgId)
  - dummy call
 * List<TaskoBunch> listBunches(Integer orgId) {
  - lists all available organizational bunches
 * Date scheduleBunch(Integer orgId, String bunchName, String jobLabel, Date startTime, Date endTime, String cronExpression, Map params) throws NoSuchBunchTaskException, InvalidParamException
  - start scheduling a organizational bunch
 * Date scheduleBunch(Integer orgId, String bunchName, String jobLabel, String cronExpression, Map params) throws NoSuchBunchTaskException, InvalidParamException
  - start scheduling a organizational bunch
 * Integer unscheduleBunch(Integer orgId, String jobLabel) throws InvalidParamException
  - stop scheduling an organizational bunch
 * Date scheduleSingleBunchRun(Integer orgId, String bunchName, Map params, Date start) throws NoSuchBunchTaskException, InvalidParamException
  - schedule a one time organizational bunch
 * Date scheduleSingleBunchRun(Integer orgId, String bunchName, Map params) throws NoSuchBunchTaskException, InvalidParamException
  - schedule a one time organizational bunch asap
 * List<TaskoSchedule> listAllSchedules(Integer orgId)
  - lists all organizational schedules
 * List<TaskoSchedule> listActiveSchedules(Integer orgId)
  - lists all active organizational schedules
 * List<TaskoSchedule> listActiveSchedulesByBunch(Integer orgId, String bunchName) throws NoSuchBunchTaskException
  - lists all active organizational schedules associated with given bunch
 * List<TaskoRun> listScheduleRuns(Integer orgId, Integer scheduleId)
  - lists all organizational runs of a give schedule
 * String getRunStdOutputLog(Integer orgId, Integer runId, Integer nBytes) throws InvalidParamException
  - get last specified number of bytes of the organizational run std output log, whole log is returned if nBytes is negative
 * String getRunStdErrorLog(Integer orgId, Integer runId, Integer nBytes) throws InvalidParamException
  - get last specified number of bytes of the organizational run std error log, whole log is returned if nBytes is negative

*Note:* To call the methods over tomcat, make sure you replace the first orgId parameter with sessionKey.