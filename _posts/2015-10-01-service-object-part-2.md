---
layout: post
title: Service Object Part 2
categories: [service objects, rails]
tags: [poro, rails, service objects]
published: True

---

Service Objects have many strengths, but one of its weakness comes directly from its strength - `Flexibility`. A SericeObject has no real structure, it could be something as simple a Plain Old Ruby Object (PORO) or a complex class with an action method which extends another class. It could have one public action method  or five. It could return errors in any way it wishes. It could handle results however it wants. To sum it up, it lacks structure and opinion. It could do things however it wishes and still be called a ServiceObject.

Well I see issues in that. There needs to be some structure, otherwise you have all these differnt things all hiding under the name of Service Objects. Below are examples of how I attempted to address this issue by giving ServiceObjects structure and opinion.

### Class

An example of a convention heavy, opinionated Service Object

````
# its placed in the services folder
# /app/services/adjust_user_target_start_points_service.rb

# the name is a verb/action with with *_service.rb pattern
class AdjustUserTargetStartPointsService

  def self.build(user_id)

    # setting up dependencies here
    user = User.find_by_id user_id

    #  injecting dependencies here
    self.new(user)
  end

  def initialize(user)

    # storing input in instance variables
    # there is no attr_reader so its not available to the outside world for access
    # there is also no attr_writer so the object is immutable
    @user = user

  end

  def call

    # do stuff here

    return ServiceResult.new :status => true, :message => "Analysis Complete"

  rescue => e

    # log the error here
    return ServiceResult.new :status => false, :message => "Error occurred - #{e.message}"

  end

  private

  # private helper methods go here

end
````

### Result

The service object aboce returns a `ServiceResult` object. Here is an example of that class.

````
class ServiceResult

  # no attr_writer here coz the result is immutable
  attr_reader :status, :message, :data, :errors

  def initialize(status:, message: nil, data: nil, errors: [])
    @status = status
    @message = message
    @data = data
    @errors = errors
  end

  def success?
    status == true
  end

  def failure?
    !success?
  end

  def has_data?
    data.present?
  end

  def has_errors?
    errors.present? && errors.length > 0
  end

  def to_s
    "#{success? ? 'Success!' : 'Failure!'} - #{message} - #{data}"
  end

end
````

### Test

Here is an example of a spec sheet to make sure that our service is built how we expect it to be.

````
describe AdjustUserTargetStartPointsService do

  context "is a service" do
    before(:each) do
      @user = FactoryGirl.create :user
      @servObj = AdjustUserTargetStartPointsService.new(@user)
    end

    it "has .build method" do
      expect(AdjustUserTargetStartPointsService).to respond_to(:build)
    end

    it "has #call method" do
      expect { @servObj.call }.to_not raise_error
    end

    it "return a ServiceResult object" do
      expect(@servObj.call.class).to eq(ServiceResult)
    end

    it "never raises an error" do
      @servObj = AdjustUserTargetStartPointsService.new(nil)
      expect { @servObj.call }.to_not raise_error
    end

    it "returns ServiceResult with false status when an error happens" do
      @servObj = AdjustUserTargetStartPointsService.new(nil)
      result = @servObj.call
      expect(result.class).to eq(ServiceResult)
      expect(result.status).to eq(false)
    end

    it "has only 1 instance method named `call` " do
      methods = @servObj.class.instance_methods - @servObj.class.parent.instance_methods
      expect(methods.length).to eq(1)
      expect(methods[0]).to eq(:call)
    end

    it "has only 1 class method named `build` " do
      methods = @servObj.class.methods - @servObj.class.parent.methods
      expect(methods.length).to eq(1)
      expect(methods[0]).to eq(:build)
    end
  end
end

````

### Conclusion

With the above three things in place we are off to a great start towards build opinionated, convention drive service objects.