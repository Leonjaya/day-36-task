# day-36-task
db.topics.find({ taught_on: { $gte: new Date("2020-10-01"), $lt: new Date("2020-11-01") } });

db.tasks.find({ submitted_on: { $gte: new Date("2020-10-01"), $lt: new Date("2020-11-01") } });
db.company_drives.find({ 
    drive_date: { 
        $gte: new Date("2020-10-15"), 
        $lte: new Date("2020-10-31") 
    } 
});
db.users.aggregate([
    {
        $lookup: {
            from: "company_drives",
            localField: "user_id",
            foreignField: "user_id",  // assuming there's a user_id in company_drives collection to map
            as: "company_drive_info"
        }
    },
    {
        $unwind: "$company_drive_info"
    },
    {
        $project: {
            user_name: "$name",
            company_name: "$company_drive_info.company_name",
            drive_date: "$company_drive_info.drive_date"
        }
    }
]);
db.codekata.aggregate([
    { $match: { status: "solved" } },
    { $group: { _id: "$user_id", problems_solved: { $sum: 1 } } }
]);
db.mentorships.aggregate([
    { $group: { _id: "$mentor_id", mentee_count: { $sum: 1 } } },
    { $match: { mentee_count: { $gt: 15 } } },
    { $lookup: {
        from: "mentors",
        localField: "_id",
        foreignField: "mentor_id",
        as: "mentor_info"
    }},
    { $unwind: "$mentor_info" },
    { $project: { mentor_name: "$mentor_info.name", mentee_count: 1 } }
]);
db.attendance.aggregate([
    { $match: { date: { $gte: new Date("2020-10-15"), $lte: new Date("2020-10-31") }, status: "absent" } },
    { $lookup: {
        from: "tasks",
        localField: "user_id",
        foreignField: "assigned_to",
        as: "task_info"
    }},
    { $unwind: "$task_info" },
    { $match: { "task_info.status": "not submitted" } },
    { $group: { _id: "$user_id", count: { $sum: 1 } } },
    { $count: "total_users" }
]);

