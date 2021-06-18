# PDD

## The problem

### Unmanageable project

Inability of a manager to manage a task or a programmer if size of tasks is too
big or undefined. Time the task takes is a week or even more.

### How to break down big tasks?

How to break down big tasks into **small** tasks. It is up to managers or
analytics. We usually don't have that kind of managers/analytics who can
decompose tasks from technical point of view.

## How to solve both problems

1. Give a programmer a large task
2. Give him only 30 minutes for it
3. Allow him to be lazy and leave the task partially unaccomplished

## What should programmer do?

1. Solve the problem at the highest level you can
2. Mark not implemented parts with `# TODO/178485955` and describe what needs
    to be done
3. Create a PR that passes your build pipeline (linter, tests, code review)
4. When it is merged to master branch, create a task for all to-do items

You go from very large abstract tasks that will produce lots of puzzles to
very small and concrete ones that won't have any puzzles inside.

## Puzzle format

1. It should have an indicator of a puzzle, e. g. `TODO` or whatever you want
2. It should reference a parent task (what task you were doing when you added
    this puzzle)
3. It should have a description

## The tree of tasks

        #1 Implement the online shop
        /                      \
  #2 Develop a web server      #3 Develop GUI
  /        \                      /       \
  ...                                ...
  /
 ✅ #345 Post a review
  /
✅ #348 Persist review in db

Having a few trees a manager knows how much time some feature takes.

## An example

I'm given a task:
`#392: Email users XSLX reports with their activities.`

```ruby
# TODO/392:
# Perform sending emails in the other background process
class ReportsController < ApplicationController
  def trigger
    User.all.find_each do |user|
      ReportMailer.with(user).deliver_now
    end
  end
end

class ReportMailer < ApplicationMailer
  def user_activities(user)
    attachment['my_activities.xlsx'] = UserActivitiesReport.new(user).to_xslx
    mail(to: user.email, subject: 'Activities')
  end
end

# TODO/392:
# generate user activities report and write it to the excel document.
class UserActivitiesReport
  def initialize(_user); end

  def to_xslx
    RubyXL::Workbook.new.stream
  end
end
```

## Concerns

### I am not a manager/don't want to create tasks/want to code only/Automation

First, you don't have to analyze business stuff to decompose a task. You should
be only a tech specialist since you are breaking it down from a **technical**
point of view.

Second, if you don't want to open your Pivotal Tracker every time your PR is
merged, then you should create some automation that does it for you. Strictly
define the **format** of puzzles, so that you can parse it and create PT stories
from them automatically.

### How to TDD

PDD goes very well with TDD. The best option is to create a test first,
**disable** it and provide some blank implementation. For exapmle you are given
a task to enable users to subscribe to email notifications.

```ruby
Rails.application.routes.draw do
  resource :settings, only: %i[update]
end

class SettingsController < ApplicationController
  def update
    render json: {}
  end
end

RSpec.describe SettingsController do
  # TODO/106084363:
  # Enable this test by implementing SettingsController#update action.
  # Estimate: 30 minutes
  # Role: Dev
  xcontext 'when user signed in' do
    before { sign_in user }

    let(:user) { create(:user) }

    context 'when sending email = true' do
      it 'subscribes user to email notifications' do
        post settings_path, settings: { notifications: { email: true } }

        expect(user).to have_email_notifications_enabled
      end
    end
  end

  context 'when user is not signed in' do
    it_behaves_like 'an authorized resource'
  end
end
```

### Pros

+ It is supposed to be more predictable
+ Less "stupid" tasks
+ You deliver results faster
+ Better competence distribution
+ You can measure performance
+ More attention in code review
+ It is very agile
+ It is fun and interesting

### Cons

- More bureaucracy
- More feature flagging to avoid unimplemented features in production
- Easy to overuse it
- **You tell me**
