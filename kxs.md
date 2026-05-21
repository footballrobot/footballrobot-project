第一周~第七周
自学Unity、c#、强化学习、足球规则等基础知识
第八周：
确认个人任务目标，着手策划机器人守门员的训练，并且列出详细的计划清单
第九周：
排障（由于版本与插件存在严重的不兼容问题，排障过程异常地长。）
第十周：
完成球场坐标标定以及范围限制，设计完毕随机发球程序（直球）。球会从球场内任意一点以任意速度发出，飞向球门。
设计完毕随机发球程序（弧线球）。由于目前的机器人功能有限，我没有使用这个程序进行测试。后续的精细化的情况下可能会使用。
随机发球程序（旋转球），可以在空中沿xyz三个方向偏移，且触碰到球门门框后有概率滚入球门。此功能正在开发中
第十一周：
研究g1机器人物理机制以及碰撞箱设置，初步训练
第十二周~第十三周：
训练机器人在球场上正常行走、横向移动、转弯，并且跟踪球的坐标。足够接近球后会触发扑球动作。
训练机制大致是：奖励：机器人成功阻止球飞向球门 惩罚：1.未能阻止球飞向球门 2.接近球之前即摔倒 3.任何情况下后仰摔倒
行为逻辑代码（片段）：
   // 应用追踪控制（如果启用）
    if (enableGoalkeeperMode && enableExplicitTracking && ballFired && !gkEpisodeEnding)
    {
        ApplyExplicitTrackingAndHandControl();
    }

    // 执行关节控制
    for (int i = 0; i < ActionNum && i < acts.Length; i++)
    {
        if (acts[i] != null)
            SetJointTargetDeg(acts[i], utotal[i]);
    }

    if (BipedTargetMotion != LastBmotion || QuadrupedTargetMotion != LastQmotion || LegwheelTargetMotion != LastLmotion) EndEpisode();
    LastBmotion = BipedTargetMotion;
    LastQmotion = QuadrupedTargetMotion;
    LastLmotion = LegwheelTargetMotion;
}

// ==================== 守门员追踪：转向 + 前进拦截 ====================
// ==================== 守门员追踪：转向 + 前进 + 急转拦截 ====================
void ApplyExplicitTrackingAndHandControl()
{
    if (ballTransform == null || body == null || !ballFired || gkEpisodeEnding) return;

    // 1. 获取球在机器人局部坐标系中的位置
    Vector3 ballLocal = body.InverseTransformPoint(ballTransform.position);
    float ballZ = ballLocal.z;      // 横向偏移：+左，-右
    float ballX = ballLocal.x;      // 前后距离：+前，-后（注意：InverseTransformPoint 中，+x 是机器人前方）

    // 关键修正：判断球是否在背后
    // 如果 ballX < 0，球在机器人后方（因为 InverseTransformPoint 的 x 轴指向机器人前方）
    bool ballBehind = ballX < -0.5f;  // 球在机器人后方 0.5m 以上
    bool ballSideBack = Mathf.Abs(ballZ) > 1.0f && ballX < 0;  // 侧后方

    // 2. 计算到球的距离和方向
    float distToBall = Mathf.Sqrt(ballX * ballX + ballZ * ballZ);
    float angleToBall = Mathf.Atan2(ballZ, -ballX) * Mathf.Rad2Deg;

    Debug.Log($"[{name}] 追踪: ballZ={ballZ:F2}, ballX={ballX:F2}, " +
              $"dist={distToBall:F2}, angle={angleToBall:F1}, behind={ballBehind}");

    // 3. 背后球处理：急转 180° + 后退
    if (ballBehind)
    {
        // 急转：通过 hip yaw 差分实现快速转身
        float spinDirection = Mathf.Sign(ballZ);  // 球在左则左转，在右则右转
        
        // hip yaw 快速旋转（如果索引有效）
        if (leftHipYawIdx >= 0 && leftHipYawIdx < ActionNum)
            utotal[leftHipYawIdx] += spinDirection * 45f;  // 大幅度旋转
        
        if (rightHipYawIdx >= 0 && rightHipYawIdx < ActionNum)
            utotal[rightHipYawIdx] -= spinDirection * 45f; // 反向旋转产生扭矩

        // 后退步：反向 hip pitch
        if (leftHipPitchIdx >= 0 && leftHipPitchIdx < ActionNum)
            utotal[leftHipPitchIdx] -= 20f;  // 后退
        
        if (rightHipPitchIdx >= 0 && rightHipPitchIdx < ActionNum)
            utotal[rightHipPitchIdx] -= 20f;

        // 身体前倾准备向后扑救（反向）
        if (waistIdx >= 0 && waistIdx < ActionNum)
            utotal[waistIdx] += spinDirection * 30f;  // 腰部扭转辅助转身

        return;  // 背后球优先处理，不走正常追踪逻辑
    }

    // 4. 正常前方/侧方追踪：转向 + 前进
    float turnAmount = Mathf.Clamp(angleToBall * 1.5f, -25f, 25f);
    float forwardAmount = 0f;

    if (distToBall > 2.0f) forwardAmount = 15f;
    else if (distToBall > 1.0f) forwardAmount = 8f;
    else forwardAmount = 0f;

    // 应用到 hip pitch
    if (leftHipPitchIdx >= 0 && leftHipPitchIdx < ActionNum)
        utotal[leftHipPitchIdx] += forwardAmount - turnAmount;
    
    if (rightHipPitchIdx >= 0 && rightHipPitchIdx < ActionNum)
        utotal[rightHipPitchIdx] += forwardAmount + turnAmount;

    // 5. 近距离扑救准备
    if (distToBall < 1.5f)
    {
        if (leftHipPitchIdx >= 0 && leftHipPitchIdx < ActionNum)
            utotal[leftHipPitchIdx] += 10f;
        if (rightHipPitchIdx >= 0 && rightHipPitchIdx < ActionNum)
            utotal[rightHipPitchIdx] += 10f;
            
        if (leftKneeIdx >= 0 && leftKneeIdx < ActionNum)
            utotal[leftKneeIdx] -= 8f;
        if (rightKneeIdx >= 0 && rightKneeIdx < ActionNum)
            utotal[rightKneeIdx] -= 8f;
    }

    // 6. 腰部辅助转向
    if (waistIdx >= 0 && waistIdx < ActionNum)
    {
        float waistTurn = Mathf.Clamp(angleToBall * 0.3f, -15f, 15f);
        utotal[waistIdx] += waistTurn;
    }

    // 7. hip roll 保持 0（避免侧向摔倒）
    if (leftHipRollIdx >= 0 && leftHipRollIdx < ActionNum)
        utotal[leftHipRollIdx] = 0;
    if (rightHipRollIdx >= 0 && rightHipRollIdx < ActionNum)
        utotal[rightHipRollIdx] = 0;
}




    // =====================================================================

void SetJointTargetDeg(ArticulationBody joint, float x)
{
    var drive = joint.xDrive;
    drive.target = x;
    joint.xDrive = drive;
}
void SetJointTargetPosition(ArticulationBody joint, float x)
{
    x = (x + 1f) * 0.5f;
    var x1 = Mathf.Lerp(joint.xDrive.lowerLimit, joint.xDrive.upperLimit, x);
    var drive = joint.xDrive;
    
    string n = joint.name.ToLower();
    bool isArm = n.Contains("shoulder") || n.Contains("elbow") || 
                 n.Contains("wrist") || n.Contains("hand") || n.Contains("arm");
    
    if (isArm)
    {
        drive.stiffness = 50f;
        drive.damping = 10f;
        drive.forceLimit = 15f;
    }
    else
    {
        drive.stiffness = 2000f;
        drive.damping = 100f;
        drive.forceLimit = 200f;
    }
    
    drive.target = x1;
    joint.xDrive = drive;
}

    public override void Heuristic(in ActionBuffers actionsOut)
    {
    }

    void FixedUpdate()
    {
        if (body == null || arts[0] == null)
        {
            return;
        }

        tp++;
        tq++;
        tt++;
        if (tp > 0 && tp <= T1)
        {
            tp0 = tp;
            uf1 = (-Mathf.Cos(3.14f * 2 * tp0 / T1) + 1f) / 2f;
            uf2 = 0;
        }
        if (tp > T1 && tp <= 2 * T1)
        {
            tp0 = tp - T1;
            uf1 = 0;
            uf2 = (-Mathf.Cos(3.14f * 2 * tp0 / T1) + 1f) / 2f;
        }
        if (tp >= 2 * T1) tp = 0;
        uff = (-Mathf.Cos(3.14f * 2 * tq / T2) + 1f) / 2f;
        if (tq >= T2) tq = 0;

        var vel = body.InverseTransformDirection(arts[0].velocity);
        var wel = body.InverseTransformDirection(arts[0].angularVelocity);

        float reward;

        if (!enableGoalkeeperMode)
        {
            var live_reward = 1f;
            var ori_reward1 = -0.1f * Mathf.Abs(EulerTrans(body.eulerAngles[0]));
            var ori_reward2 = -2f * Mathf.Abs(wel[1]);
            var ori_reward3 = -0.1f * Mathf.Min(Mathf.Abs(body.eulerAngles[2]), Mathf.Abs(body.eulerAngles[2] - 360f));
            var vel_reward2 = vel[2] - Mathf.Abs(vel[0]) + kh * Mathf.Abs(vel[1]);
            reward = live_reward + (ori_reward1 + ori_reward2 + ori_reward3) * ko + vel_reward2;
        }
        else
        {
            float pelvisHeight = body.position.y;
            float pitch = EulerTrans(body.eulerAngles[0]);
            float roll = EulerTrans(body.eulerAngles[2]);

            float height_reward;
            if (pelvisHeight < 0.25f)
                height_reward = -5.0f;
            else if (pelvisHeight < 0.35f)
                height_reward = -2.0f;
            else if (pelvisHeight < 0.45f)
                height_reward = 1.0f;
            else
                height_reward = 3.0f;

            float ori_penalty = -0.2f * Mathf.Abs(pitch) - 0.2f * Mathf.Abs(roll);
            float vel_penalty = -0.3f * arts[0].velocity.magnitude;
            float live_reward = 0.5f;

            float look_reward = 0f;
            if (pelvisHeight > 0.4f && ballTransform != null)
            {
                Vector3 toBall = (ballTransform.position - body.position).normalized;
                float facingBall = Vector3.Dot(body.forward, toBall);
                look_reward = gkRewardLookAtBall * Mathf.Max(0, facingBall);
            }

            reward = live_reward + height_reward + ori_penalty + vel_penalty + look_reward;
        }

        AddReward(reward);

        if (enableGoalkeeperMode && ballFired && !gkEpisodeEnding)
        {
            gkEpisodeTimer += Time.fixedDeltaTime;

            if (ballTransform != null)
            {
                ballRelativePos = body.InverseTransformPoint(ballTransform.position);
                ballRelativeVel = body.InverseTransformDirection(ballVelocity);

                if (ballSpeed > 0.01f)
                {
                    float distToGoal = Mathf.Abs(ballTransform.position.x - goalX);
                    timeToArrive = distToGoal / ballSpeed;
                    Vector3 predictedLanding = ballTransform.position + ballVelocity * timeToArrive;
                    predictedLandingRelative = body.InverseTransformPoint(predictedLanding);
                }

                float pelvisHeight = body.position.y;
                if (pelvisHeight > 0.35f)
                {
                    Vector3 robotPosXZ = new Vector3(body.position.x, 0, body.position.z);
                    Vector3 landingXZ = new Vector3(ballTargetPos.x, 0, ballTargetPos.z);
                    float distXZ = Vector3.Distance(robotPosXZ, landingXZ);
                    AddReward(gkRewardApproach * Mathf.Exp(-distXZ));
                }

                if (ballTransform.position.x <= goalX + 0.05f)
                {
                    gkEpisodeEnding = true;
                    bool isGoal = ballTransform.position.y >= goalYRange.x &&
                                  ballTransform.position.y <= goalYRange.y &&
                                  ballTransform.position.z >= goalZRange.x &&
                                  ballTransform.position.z <= goalYRange.y;
                    if (isGoal)
                    {
                        AddReward(gkRewardGoal);
                        Debug.Log($"[{this.name}] 失球");
                    }
                    else
                    {
                        AddReward(gkRewardSave);
                        Debug.Log($"[{this.name}] 射偏/保存");
                    }
                    if (train) EndEpisode();
                }

                if (gkEpisodeTimer >= gkMaxEpisodeTime)
                {
                    gkEpisodeEnding = true;
                    if (train) EndEpisode();
                }

                if (ballTransform.position.x < goalX - 2f ||
                    Mathf.Abs(ballTransform.position.z) > 4f ||
                    ballTransform.position.y > 3f)
                {
                    gkEpisodeEnding = true;
                    if (train) EndEpisode();
                }
            }
        }

        SphereCastBlockDetection();

        float maxTilt = enableGoalkeeperMode ? 45f : 20f;
        float fallPitch = EulerTrans(body.eulerAngles[0]);
        float fallRoll = EulerTrans(body.eulerAngles[2]);
        bool isFallen = Mathf.Abs(fallPitch) > maxTilt || Mathf.Abs(fallRoll) > maxTilt;

        if (isFallen || tt >= 1000)
        {
            if (train)
            {
                // 守门员模式：根据与球距离决定是否惩罚摔倒
                if (enableGoalkeeperMode && isFallen)
                {
                    float distToBall = float.MaxValue;
                    if (ballTransform != null && ballFired)
                        distToBall = Vector3.Distance(body.position, ballTransform.position);

                    if (distToBall > fallPenaltyDistance)
                    {
                        AddReward(fallPenaltyFar);
                        Debug.Log($"[{name}] 远离球摔倒 (距离 {distToBall:F2}m)，惩罚 {fallPenaltyFar}");
                    }
                    else
                    {
                        if (fallPenaltyClose != 0f)
                            AddReward(fallPenaltyClose);
                        Debug.Log($"[{name}] 近距离扑救摔倒 (距离 {distToBall:F2}m)，豁免惩罚");
                    }
                }
                EndEpisode();
            }
        }
    }
